# CML with cloud compute


This repository contains a sample project using [CML](https://github.com/iterative/cml) with [Docker Machine](https://docs.docker.com/machine/overview/) to launch an AWS EC2 instance and then run a neural style transfer on that instance. On a pull request, the following actions will occur:
- GitHub will deploy a runner with a custom CML Docker image
- Docker Machine will provision an EC2 instance and pass the neural style transfer workflow to it. DVC is used to version the workflow and dependencies. 
- Neural style transfer will be executed on the EC2 instance 
- CML will report results of the style transfer as a comment in the pull request. 

The key file enabling these actions is `.github/workflows/cml.yaml`.

