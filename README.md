# allcomponets-Github-Actions
Task:
-currenty we are deploying python app docker image pusinging to ecr after deploying to ec2 instance and we are using ec2 instance as github actions slef hosted runner.  
-using all github  actions comnentes we are using in this  deployment configuration file such as,  
**-****************  
Secrets / Environment Variables – store sensitive data like AWS keys.  
Jobs & Steps – define stages like build, push, deploy.  
Matrix Builds – run on multiple versions/OS in parallel.  
Artifacts & Caching – save build outputs, speed up builds.  
Reusable Workflows – call one workflow from another.  
Conditional Jobs – run only on main or certain paths.  
Manual Approvals – via GitHub Environments.  
Notifications – Slack, Teams, Email.  
Debugging / Logs – ACTIONS_STEP_DEBUG, workflow visualization.  
###################
steps:

#ec2 instance# 

-creating ec2 instance vpc and public private subnets and enabled public subnte and public ip for ec2 instance.

-created ecr (elastic container registry in the aws) .


-created Iam role in the IAM section with selected role and aws service ec2 and added policy as awsecrreadonly access then save that role.

-need to attach that role to existing ec2 using aws cli

-or while creatingec2 we can attch in the ui but after creation we cant add directly in the ui page in ec2 need to use only aws cli to attch role to existing ec2.

-once added now  ec2 can access the ecr to pull the image from ecr to  ec2 deployment.

cmd:  
**aws ec2 associate-iam-instance-profile \
    --instance-id i-0123456789abcdef0 \
    --iam-instance-profile Name=YourIAMRoleName**


##self hosted runner##

-go to github actions page and setting sunder that actions nad runners self  hsted runner then add the self hosted runner ,select the what type of os we are deployed in the 

ec2 then select.

-it will provide the seteps to attch the runer to github actions based on the OS.


- login into ec2 then run the above script to connect the runner to gitbu actions.

- will use ./svs.sh to automating run the runner

- sudo .svc.sh install,  start,stop


  ##environment variables and secrets##  
**IMAGE_TAG	${{ github.sha }}	Unique tag for each commit  
AWS_REGION	ap-south-1 (or your region)	Region of ECR  
ECR_REPOSITORY	my-python-app	Your ECR repository name  
#secrets##
Secret Name	Value	Notes  
AWS_ACCESS_KEY_ID	<your IAM user access key>	For GitHub Actions to push to ECR  
AWS_SECRET_ACCESS_KEY	<your IAM user secret>	For GitHub Actions to push to ECR  
AWS_ACCOUNT_ID	<12-digit AWS account ID>	Used in Docker image tag  
SONAR_PROJECT_KEY	<your SonarQube project key>	For Sonar scan  
SONAR_TOKEN	<your SonarQube token>	For Sonar scan authentication  
EC2_PUBLIC_IP	<your EC2 public IP>	For SSH deployment  
EC2_SSH_KEY	<your private SSH key>	For SSH access to EC2  
SLACK_CHANNEL_ID	<channel ID>	For Slack notifications  
SLACK_BOT_TOKEN	<Slack bot token>	For posting messages**


######  
- for environment variables we are collected aws regions is east-us-1 from aws ec2 instance and ecr repository name picked from ecr dahsboard.
- aws access key  and secret key generatedin aws profile security option..
- account id updated from aws  iam role user profile.
- for sonar  project key we need to login sonarqube dahsboard  and login  with github repository and select our repo or all repos and login then create the project. then will get project key from
dashboard and for token from need to security and gereate the token will sue in piplein yaml.  
-for security scan for docker image we are using trivy and for trivy we dont have any particular dashboard for trivy and any tokens not there. directly it will scan the images while  running th pipeline  and scan vulnerabilities and will get he output logs from  the pipleine.  
-for slack credentials need to login first slack and create the channel(workspace), once done login into slack api with salck and select our namespace after that go to settng sin that need to get bot chat read option select then install the bot app in he salck once install you will get slack bot token justcopy that one, now you got slack channel id and token.  
-now we are completed all requirements for  project setup.  

###Project deployment steps### 

Workflow Overview
name: Full Python App CI/CD

on:
  push:
    branches:
      - main
  workflow_dispatch:


Explanation:

The workflow triggers automatically on push to main branch.

workflow_dispatch allows manual runs from GitHub UI.

env:
  AWS_REGION: ${{ vars.AWS_REGION }}
  ECR_REPOSITORY: ${{ vars.ECR_REPOSITORY }}
  IMAGE_TAG: ${{ github.sha }}


Explanation:

Sets environment variables globally for all jobs.

AWS_REGION and ECR_REPOSITORY are read from GitHub variables.

IMAGE_TAG is set to the commit SHA for versioning Docker images.

Job 1: build-and-scan
build-and-scan:
  name: Build, Scan & Test
  runs-on: ubuntu-latest


Explanation:

Job runs on Ubuntu GitHub runner.

Name indicates it handles build, scanning, and testing.

Step 1: Checkout code
- name: Checkout Code
  uses: actions/checkout@v4


Explanation:

Pulls the repository code into the runner so you can build/test/deploy.

Step 2: Set up Python
- name: Set up Python 3.11
  uses: actions/setup-python@v4
  with:
    python-version: 3.11


Explanation:

Installs Python 3.11 on the runner.

Allows running Python commands and installing dependencies.

Step 3: Cache pip dependencies
- name: Cache pip dependencies
  uses: actions/cache@v3
  with:
    path: ~/.cache/pip
    key: ${{ runner.os }}-pip-${{ hashFiles('**/requirements.txt') }}
    restore-keys: |
      ${{ runner.os }}-pip-


Explanation:

Stores pip packages cache to speed up future runs.

Cache key is based on requirements.txt hash, so it updates if dependencies change.

Step 4: Install dependencies
- name: Install dependencies
  run: |
    python -m pip install --upgrade pip
    pip install -r requirements.txt


Explanation:

Upgrades pip.

Installs all Python dependencies from requirements.txt.

Step 5: Login to AWS ECR
- name: Login to AWS ECR
  uses: aws-actions/configure-aws-credentials@v4
  with:
    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    aws-region: ${{ env.AWS_REGION }}


Explanation:

Configures AWS CLI with credentials to interact with ECR and other AWS services.

Step 6: Build Docker Image
- name: Build Docker Image
  run: docker build -t ${{ env.ECR_REPOSITORY }}:${{ env.IMAGE_TAG }} .


Explanation:

Builds Docker image with your Python app.

Tags image with repository name and commit SHA.

Step 7: Push Docker Image to ECR
- name: Push Docker Image to ECR
  run: |
    aws ecr get-login-password --region ${{ env.AWS_REGION }} | docker login --username AWS --password-stdin ${{ secrets.AWS_ACCOUNT_ID }}.dkr.ecr.${{ env.AWS_REGION }}.amazonaws.com
    docker tag ${{ env.ECR_REPOSITORY }}:${{ env.IMAGE_TAG }} ${{ secrets.AWS_ACCOUNT_ID }}.dkr.ecr.${{ env.AWS_REGION }}.amazonaws.com/${{ env.ECR_REPOSITORY }}:${{ env.IMAGE_TAG }}
    docker push ${{ secrets.AWS_ACCOUNT_ID }}.dkr.ecr.${{ env.AWS_REGION }}.amazonaws.com/${{ env.ECR_REPOSITORY }}:${{ env.IMAGE_TAG }}


Explanation:

Logs in to AWS ECR.

Tags Docker image for ECR repository.

Pushes image to ECR.

Step 8: Trivy Security Scan
- name: Trivy Security Scan
  uses: aquasecurity/trivy-action@0.28.0
  with:
    image-ref: ${{ secrets.AWS_ACCOUNT_ID }}.dkr.ecr.${{ env.AWS_REGION }}.amazonaws.com/${{ env.ECR_REPOSITORY }}:${{ env.IMAGE_TAG }}


Explanation:

Runs Trivy security scan on the Docker image to detect vulnerabilities.

Step 9: SonarCloud Scan
- name: SonarCloud Scan
  uses: SonarSource/sonarcloud-github-action@v2.2.0
  env:
    SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
  with:
    args: >
      -Dsonar.projectKey=${{ secrets.SONAR_PROJECT_KEY }}
      -Dsonar.organization=sudhakar9666
      -Dsonar.host.url=https://sonarcloud.io
      -Dsonar.login=${{ secrets.SONAR_TOKEN }}


Explanation:

Performs code quality scan with SonarCloud.

Uses project key and token from GitHub secrets.

Step 10: Upload Build Artifacts
- name: Upload Build Artifacts
  uses: actions/upload-artifact@v4
  with:
    name: docker-image-info
    path: image-info.txt


Explanation:

Uploads artifact (image-info.txt) so it can be downloaded later or used by other jobs.

Job 2: deploy
deploy:
  name: Deploy to EC2 (Manual Approval)
  needs: build-and-scan
  runs-on: ubuntu-latest
  environment:
    name: production


Explanation:

Job depends on build-and-scan.

Deploys to EC2 using manual approval via GitHub environment protection rules.

Step 1: Deploy Docker Image to EC2
- name: Deploy Docker Image to EC2
  uses: appleboy/ssh-action@v0.1.6
  with:
    host: ${{ secrets.EC2_PUBLIC_IP }}
    username: ubuntu
    key: ${{ secrets.EC2_SSH_KEY }}
    script: |
      aws ecr get-login-password --region ${{ env.AWS_REGION }} | \
      docker login --username AWS --password-stdin ${{ secrets.AWS_ACCOUNT_ID }}.dkr.ecr.${{ env.AWS_REGION }}.amazonaws.com
      docker pull ${{ secrets.AWS_ACCOUNT_ID }}.dkr.ecr.${{ env.AWS_REGION }}.amazonaws.com/${{ env.ECR_REPOSITORY }}:${{ env.IMAGE_TAG }}
      docker stop my-python-app || true
      docker rm my-python-app || true
      docker run -d --name my-python-app -p 80:5000 ${{ secrets.AWS_ACCOUNT_ID }}.dkr.ecr.${{ env.AWS_REGION }}.amazonaws.com/${{ env.ECR_REPOSITORY }}:${{ env.IMAGE_TAG }}


Explanation:

Connects to EC2 via SSH using private key.

Logs in to ECR on EC2.

Pulls the latest Docker image.

Stops & removes old container if exists.

Runs a new Docker container with your app.

Step 2 & 3: Slack Notifications
- name: Slack Notification (Success)
  if: success()
  uses: slackapi/slack-github-action@v1.23.0
  with:
    channel-id: ${{ secrets.SLACK_CHANNEL_ID }}
    slack-message: "✅ Deployment completed successfully on ${{ env.IMAGE_TAG }}!"
    slack-bot-token: ${{ secrets.SLACK_BOT_TOKEN }}

- name: Slack Notification (Failure)
  if: failure()
  uses: slackapi/slack-github-action@v1.23.0
  with:
    channel-id: ${{ secrets.SLACK_CHANNEL_ID }}
    slack-message: "❌ Deployment failed on ${{ env.IMAGE_TAG }}!"
    slack-bot-token: ${{ secrets.SLACK_BOT_TOKEN }}


Explanation:

Sends Slack notifications depending on deployment result.

success() triggers if all previous steps succeeded.

failure() triggers if any step failed.

Summary of workflow steps
Job	Step	Purpose
build-and-scan	Checkout Code	Get repo code on runner
build-and-scan	Set up Python	Install Python runtime
build-and-scan	Cache pip dependencies	Speed up dependency installs
build-and-scan	Install dependencies	Install packages from requirements.txt
build-and-scan	AWS ECR Login	Configure AWS CLI to push Docker
build-and-scan	Build Docker image	Build container with app
build-and-scan	Push Docker image	Push image to ECR
build-and-scan	Trivy Scan	Security scan on Docker image
build-and-scan	SonarCloud Scan	Code quality scan
build-and-scan	Upload Artifact	Save build info
deploy	SSH Deploy	Pull image & run container on EC2
deploy	Slack Success	Notify success in Slack
deploy	Slack Failure	Notify failure in Slack



