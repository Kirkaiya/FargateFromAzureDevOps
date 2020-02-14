# FargateFromAzureDevOps

**This is a simple ASP.NET Core (3.1) MVC app with Dockerfile for using in an Azure DevOps pipeline to build and deploy to AWS Fargate (a serverless container service)**

## I. Create an Azure DevOps Project and add sample ASP.NET solution

1. In Azure DevOps, create a new project. You can name it whatever you like. I suggest you keep the visibility set to "private".

2. After the project is done being created, select 'Repos' in the left menu. You should now see options for adding code to your project.  

3. Under **Import a repository**, click the 'Import' button.

4. In the import panel on the right, ensure **Repository type** is set to 'Git', and paste the following GitHub URL into the 'Clone URL' field:

    https://github.com/Kirkaiya/FargateFromAzureDevOps.git

5. Click the 'Import' button at the bottom of the panel to begin importing code. This process will take up to 2 minutes to complete.

### II. Create the build pipeline

1. From the 'Files' view, at the top right, click the 'Set up build' button.

2. Under **Configure your pipeline**, select the 'Starter pipeline' option (see screenshot below).
![Azure Pipeline Template list](images/azure-pipeline-1.png)

3. Under **Review your pipeline YAML**, edit the pipeline YAML file, removing the hello world step, and changing the multi-line script to do the docker build. Your file should look like this after editing:

```yaml
# Starter pipeline
# Start with a minimal pipeline that you can customize to build and deploy your code.
# Add steps that build, run tests, deploy, and more:
# https://aka.ms/yaml

trigger:
- master

pool:
  vmImage: 'ubuntu-latest'

steps:
- script: |
    echo Starting docker build
    docker build -t hellofargate .
  displayName: 'Run docker build'
```

4. Next, near the top-right of the YAML editor, click the 'Show assistant' link to show the list of tasks.

5. Type 'AWS ecr' in the 'Search tasks' box, the select 'Amazon ECR Push' in the list. This will open that task in the task pane.

6. For AWS Credentials, click the drop-down and select the connection you created earlier.

7. Select the AWS region you are working in today.

8. See the screenshot below for the rest of the values to enter. Ensure you check the box for "Create repository if it does not exist".
![AWS Push to ECR Task](images/azure-pipeline-2.png)

9. Click 'Add' to add the generated YAML to your pipeline's YAML file.

10. If necessary, use the tab key to indent the (selected) YAML task block after it is inserted. Your YAML file should look similar to the example below.

11. Click 'Save and run' to save your pipeline and kick off a build (which will also push the generated container image to Amazon ECR).

```yaml
# Starter pipeline
# Start with a minimal pipeline that you can customize to build and deploy your code.
# Add steps that build, run tests, deploy, and more:
# https://aka.ms/yaml

trigger:
- master

pool:
  vmImage: 'ubuntu-latest'

steps:
- script: |
    echo Starting docker build
    docker build -t hellofargate .
  displayName: 'Run docker build'

- task: ECRPushImage@1
  inputs:
    awsCredentials: 'YourAwsServiceConnectionName'
    regionName: 'us-west-2'
    imageSource: 'imagename'
    sourceImageName: 'hellofargate'
    repositoryName: 'hellofargaterepo'
    autoCreateRepository: true
```

### III. Create the release pipeline

TBD