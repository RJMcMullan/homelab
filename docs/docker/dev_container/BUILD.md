# 🔨 Docker-Compose Build

This guide will walk you through the steps to build the dev-container image.

## 🚀 Build Steps

### Prerequisites
Make sure the ca.crt (homelab root certificate) and the python pip requirements.txt file are in the same director as the Dockerfile


1. Go to your home directory

        cd $HOME

2. git clone <https://gitlab-docker01.internal/homelab/containers/dev.git>

3. Go to the cloned folder.

        cd $HOME/dev

4. Update the Docker file
   1. Update Image and Tag and software version tags in the gitlab-ci.yml file

5. Push code to a feature branch and check the hadolint analyze stage

6. Create a git tag so the build phase begins

7. Wait for the build phase to complete and merge the feature branch to the main branch, this will then create a gitlab release

## 📫 Contact


