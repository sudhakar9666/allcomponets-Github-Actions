# allcomponets-gitbuactions
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



