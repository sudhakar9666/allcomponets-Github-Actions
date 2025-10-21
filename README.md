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

##self hosted runner##

-go to github actions page and setting sunder that actions nad runners self  hsted runner then add the self hosted runner ,select the what type of os we are deployed in the 

ec2 then select.

-it will provide the seteps to attch the runer to github actions based on the OS.


- login into ec2 then run the above script to connect the runner to gitbu actions.

- will sue ./svs.sh to automating run the runner

- sudo .svc.sh install,  start,stop
  
