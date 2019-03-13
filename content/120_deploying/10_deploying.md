+++
title = "Deploying using Docker"
chapter = false
weight = 10
+++

Navigate to the root of our project. And build the project as a docker container. The project already contains a docker file.

```
docker build -t awscya .
```

If you like you can run the container. Its should load up at http://localhost:8466

```
docker run -p 8446:80/tcp  awscya:latest
```

My Dockerhub username is thebeebs. Substitute thebeebs for your docker username and then run the command below.
```
docker tag awscya:latest thebeebs/awscya:latest
```

Substitute thebeebs for your docker username and then run the command below.
```
docker push thebeebs/awscya:latest
```

Now that your container is on DockerHub you should be able to use fargate to run your container. 

Follow these instructions on how to run a container in fargate:

https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_GetStarted.html