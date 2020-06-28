# Micro service architecture with AWS ECS and AutoScaling

## Prerequisite

- git
- docker
- aws-cli

## Create ECR repository and push image to it

```shell
$ aws cloudformation create-stack --stack-name sampleECRRepository --template-body file://ecrRepository.yaml --no-disable-rollback --parameters ParameterKey=WebServiceRepositoryName,ParameterValue=web_service ParameterKey=WebServerRepositoryName,ParameterValue=web_server
$ aws ecr get-login --no-include-email --region ${AWS.Region}
$ docker login -u AWS -p XXXXXX.......
$ docker build -t ecr_repository_name .
$ docker tag ecr_repository_name:latest ${AWS.AccountId}.dkr.ecr.${AWS.Region}.amazonaws.com/web_service:latest
$ docker tag ecr_repository_name:latest ${AWS.AccountId}.dkr.ecr.${AWS.Region}.amazonaws.com/web_server:latest
```

Then the image will be pushed to the ECR repository.

```
$ docker push ${AWS.AccountId}.dkr.ecr.${AWS.Region}.amazonaws.com/web_service:latest
$ docker push ${AWS.AccountId}.dkr.ecr.${AWS.Region}.amazonaws.com/web_server:latest
```

## Create ECS Cluster, TaskDefinition, and Service with AutoScaling EC2 containers

`ECRRepositoryArn`, the output of sampleECRRepository, is needed to be set ContainerDefinitions.ContainerDefinitions.Image in microservices.yaml like below.
```yaml
ECSTaskDefinition:
  Type: AWS::ECS::TaskDefinition
  Properties:
    ContainerDefinitions:
      - Name: 'web_service'
        Image: '${AWS.AccountId}.dkr.ecr.${AWS.Region}.amazonaws.com/web_service:latest'
      ...
      - Name: 'web_server'
        Image: '${AWS.AccountId}.dkr.ecr.${AWS.Region}.amazonaws.com/web_server:latest'
```
Finally, create AutoScalingGroup, ECS Cluster, TaskDefinition, and Service.
```shell
$ aws cloudformation create-stack --stack-name microServices --template-body file://microservices.yaml --no-disable-rollback --capabilities CAPABILITY_NAMED_IAM
```

Now, the container instances and tasks will be running even if these instances and tasks are stopped on either accident or purpose.
