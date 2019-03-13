+++
title = "Deploying Using Elastic Beanstalk"
chapter = false
weight = 20
+++

Navigate to the root of the project in your commandline and run

```
dotnet publish AWSCYA.csproj --configuration Release --output deploy
```
This will generate a folder called deploy.

Navigate to the deploy folder and zip up the contents. The project already contains configuration files for elastic beanstalk. This zip folder is what we will upload to beanstalk.

![Deploy](/images/eb-deploy-1.png)

### Setup a Beanstalk Environment.

Follow the instuctions, ensuring you create a Windows environment and upload the zip file we created. 

https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.environments.html
