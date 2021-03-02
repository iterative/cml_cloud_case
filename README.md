# CML with cloud compute


This repository contains a sample project using [CML](https://github.com/iterative/cml) with Terraform (via the `cml-runner` function) to launch an AWS EC2 instance and then run a neural style transfer on that instance. On a pull request, the following actions will occur:
- GitHub will deploy a runner with a custom CML Docker image
- `cml-runner` will provision an EC2 instance and pass the neural style transfer workflow to it. DVC is used to version the workflow and dependencies. 
- Neural style transfer will be executed on the EC2 instance 
- CML will report results of the style transfer as a comment in the pull request. 

The key file enabling these actions is `.github/workflows/cml.yaml`.

## Variables
You must create a [personal access token](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token) with repository and workflow privileges to supply as a secret, in addition to your AWS credentials (`AWS_ACCESS_KEY_ID`,`AWS_SECRET_ACCESS_KEY`, and optinoally `AWS_SESSION_TOKEN`).
