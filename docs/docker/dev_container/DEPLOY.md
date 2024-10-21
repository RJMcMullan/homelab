# 📚 How to deploy the DEV Container

This guide will walk you through how to deploy a docker development container.

## 🚀 Deployment Steps

1. Run: docker login registry-gitlab-docker01.internal
        docker pull registry-gitlab-docker01.internal/homelab/containers/dev:latest

2. Go to your home directory

        cd $HOME

3. git clone <https://gitlab-docker01.internal/homelab/containers/dev.git>


4. Go to the cloned folder.

        cd $HOME/dev

5. Copy the sample env.example file:

        cp -R .env.example .env.user

6. Update the following environment variables in the .env.user file
   1. IMAGE_NAME
   2. IMAGE_TAG
   3. USER

7. Copy your host .gitconfig to /home/{TGI}/container_config/

        cp $HOME/.gitconfig $HOME/container_config/
 

8. Copy your host .bashrc to /home/{TGI}/container_config/

        cp $HOME/.bashrc $HOME/container_config/



9. To deploy your dev container run:

        docker-compose --env-file .env.user up -d

10. To use your dev container run:

        docker exec -it {TGI}_devcontainer /bin/bash

11. Your host working directory structure should look like the following example:

        .
        ├── home
        │   ├── {USER}
        │   │   ├── container_config
        │   │   │   |
        │   │   │   |
        │   │   │   ├── .bashrc
        │   │   │   ├── .gitconfig
        │   │   └── dev-container
        │   │   │   └── ...
        │   │   └── git
        │   │   │   └── {PROJECT-A}
        │   │   │   └── {PROJECT-B}
        │   │   │   └── ...

12. To stop your dev container run:

        docker-compose --env-file .env.user down

## 📫 Contact
