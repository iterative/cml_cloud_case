# CML with cloud compute


This repository contains a sample project using [CML](https://github.com/iterative/cml) with [Docker Machine](https://docs.docker.com/machine/overview/) to launch an AWS EC2 instance and then run a neural style transfer on that instance. On a pull request, the following actions will occur:
- GitHub will deploy a runner with a custom CML Docker image
- Docker Machine will provision an EC2 instance and pass the neural style transfer workflow to it. DVC is used to version the workflow and dependencies. 
- Neural style transfer will be executed on the EC2 instance 
- CML will report results of the style transfer as a comment in the pull request. 

The key file enabling these actions is `.github/workflows/cml.yaml`.

## Using a different cloud service
This example uses AWS EC2 instances. The workflow is compatible with Azure compute as well. However, you will need to modify the `deploy` job in `.github/workflows/cml.yaml` for GCP Compute Engine:

```yaml
deploy-gce-gpu:
    runs-on: [ubuntu-latest]
    container: docker://dvcorg/cml:latest

    steps:
      - name: deploy
        shell: bash
        env:
          repo_token: ${{ secrets.REPO_TOKEN }} 
          GOOGLE_APPLICATION_CREDENTIALS_DATA: ${{ secrets.GOOGLE_APPLICATION_CREDENTIALS_DATA }}
        run: |
          echo "Deploying..."

          echo '${{ secrets.GOOGLE_APPLICATION_CREDENTIALS_DATA }}' > gce-credentials.json
          export GOOGLE_APPLICATION_CREDENTIALS='gce-credentials.json'

          RUNNER_LABELS="gce"
          RUNNER_REPO="https://github.com/${GITHUB_REPOSITORY}"
          MACHINE="cml$(date +%s)"

          docker-machine create --driver google \
            --google-machine-type "n1-standard-4" \
            --google-project "cml-project-279709" \
            --google-machine-image "https://www.googleapis.com/compute/v1/projects/ubuntu-os-cloud/global/images/family/ubuntu-1804-lts" \
            --google-accelerator-type "nvidia-tesla-p100" \
            --google-accelerator-count 1 \
            $MACHINE

          eval "$(docker-machine env --shell sh $MACHINE)"

          (
          docker-machine ssh $MACHINE "sudo mkdir -p /docker_machine && sudo chmod 777 /docker_machine" && \
          docker-machine scp -r -q ~/.docker/machine/ $MACHINE:/docker_machine && \
          docker-machine scp -q gce-credentials.json $MACHINE:/docker_machine/gce-credentials.json && \
          docker-machine ssh $MACHINE "sudo apt update && sudo apt install -y ubuntu-drivers-common && sudo ubuntu-drivers autoinstall" && \
          docker-machine ssh $MACHINE "curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add - && curl -s -L https://nvidia.github.io/nvidia-docker/ubuntu18.04/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list && sudo apt update && sudo apt install -y nvidia-container-toolkit && \
          docker-machine restart $MACHINE && \
          eval "$(docker-machine env --shell sh $MACHINE)" && \
          docker run --name runner --gpus all -d \
            -v /docker_machine/gce-credentials.json:/gce-credentials.json \
            -e GOOGLE_APPLICATION_CREDENTIALS='/gce-credentials.json' \
            -v /docker_machine/machine:/root/.docker/machine \
            -e DOCKER_MACHINE=$MACHINE \
            -e repo_token=$repo_token \
            -e RUNNER_LABELS=$RUNNER_LABELS \
            -e RUNNER_REPO=$RUNNER_REPO \
            -e RUNNER_IDLE_TIMEOUT=120 \
            docker://dvcorg/cml-py3 && \
          sleep 20 && echo "Deployed $MACHINE"
          ) || (docker-machine rm -f $MACHINE && exit 1)
```
