# Step-by-step Implementation of CICD of ECS Fargate Deployment using Github Actions:

created a github workflow "cicd-workflow.yml" in the following repo: https://github.com/RajalakshmiNagendran/sample-app.git 

securing credentials for the pipeline: In Github as Repository secrets (Eg: AWS_ACCESS_KEY, AWS_SECRET_ACCESS_KEY)

By this way Github actions will connect with AWS 

steps involved in Github Actions for CICD:
1. Checkout source
2. Login to Amazon ECR
3. Build, tag, and push image to Amazon ECR
4. Fill in the new image ID in the Amazon ECS task definition
5. Deploy Amazon ECS task definition

Now if there is any push happened automatically 
