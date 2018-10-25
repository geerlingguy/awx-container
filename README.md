# AWX (Built with Ansible Container)

[![Build Status](https://travis-ci.org/geerlingguy/awx-container.svg?branch=master)](https://travis-ci.org/geerlingguy/awx-container) [![](https://images.microbadger.com/badges/image/geerlingguy/awx_web.svg)](https://microbadger.com/images/geerlingguy/awx_web "Get your own image badge on microbadger.com") [![](https://images.microbadger.com/badges/image/geerlingguy/awx_task.svg)](https://microbadger.com/images/geerlingguy/awx_task "Get your own image badge on microbadger.com")

This project is in its early stages. _There will be bugs!_

This project is composed of three main parts:

  - **Ansible Container project**: This project is maintained on GitHub: [geerlingguy/awx-container](https://github.com/geerlingguy/awx-container). Please file issues, support requests, etc. against this GitHub repository.
  - **Docker Hub Image**: If you just want to use [`geerlingguy/awx_web`](https://hub.docker.com/r/geerlingguy/awx_web/) and [`geerlingguy/awx_task`](https://hub.docker.com/r/geerlingguy/awx_task/) in your project, you can pull it from Docker Hub.
  - **Ansible Role**: If you need an Ansible role to build AWX, check out [`geerlingguy.awx`](https://galaxy.ansible.com/geerlingguy/awx/) on Ansible Galaxy. (This is the Ansible role that does the bulk of the work in managing the AWX container.)

## Versions

Currently maintained versions include:

  - `geerlingguy/awx_web`:
    - `1.x`, `latest`: AWX 1.x
    - `1.0.5`
  - `geerlingguy/awx_task`:
    - `1.x`, `latest`: AWX 1.x
    - `1.0.5`

## Quickstart - Standalone Usage with Docker Compose

If you just want to get an AWX environment running quickly, you can use the `docker-compose.yml` file included with this project to build a local environment accessible on `http://localhost:80/`:

    mkdir awx-test && cd awx-test
    curl -O https://raw.githubusercontent.com/geerlingguy/awx-container/master/docker-compose.yml
    docker-compose up -d

The Docker Compose file uses community images for `postgres`, `rabbitmq`, and `memcached`, and the following images for AWX:

  - [`ansible/awx_web`](https://hub.docker.com/r/ansible/awx_web/)
  - [`ansible/awx_task`](https://hub.docker.com/r/ansible/awx_task/)

After the initial database migration completes (this can take a few minutes; follow the progress with `docker logs -f [id-of-awx_task-container]`), you will be able to access the AWX interface at `http://localhost/`. The default login is `admin`/`password`.

> Note: Switch the image for the `awx_web` and `awx_task` containers in `docker-compose.yml` if you want to use the `geerlingguy/` maintained images rather than the ones from `ansible/`. If you're just kicking AWX's tires though, stick with the defaults.

## Management with Ansible Container

### Prerequisites

Before using this project to build and maintain AWX images for Docker, you need to have the following installed:

  - [Docker Community Edition](https://docs.docker.com/engine/installation/) (for Mac, Windows, or Linux)
  - [Ansible Container](https://docs.ansible.com/ansible-container/installation.html)

### Build the AWX images

Typically, you would build the images as specified in the `container.yml` file using `ansible-container --var-file config.yml build`, but in this case, since there are many dependencies bundled in the AWX repository, we will build the Docker images using a helper playbook:

    $ cd prebuild/
    $ ansible-galaxy install -r requirements.yml --force
    $ ansible-playbook -i 'localhost,' -c local prebuild.yml

After this playbook runs, you should see two new Docker images (which we'll use in the Ansible Container definition):

    $ docker images
    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
    awx_task            devel               26311794058d        29 seconds ago      938MB
    awx_web             devel               3d38dccc9190        58 seconds ago      913MB

> A Vagrantfile is included with this project to assist in building a clean environment with all the dependencies required to build the AWX images (in case you don't want to install everything on your local workstation!). To use it:
> 
>   1. Run `vagrant up`.
>   2. Wait for Vagrant's provisioning to complete (it will run `prebuild.yml` automatically).
>   3. Log in with `vagrant ssh` and use `docker` or `ansible` as needed.

### Build the conductor

Build the conductor using `ansible-container build`:

    ansible-container --var-file config.yml build

> Note: If you get any permission errors trying to generate a Docker container, make sure you're either running the commands as root or with sudo, or your user is in the `docker` group (e.g. `sudo usermod -G docker -a [user]`, then log out and log back in).

### Run the containers

    ansible-container --var-file config.yml run

You should be able to reach AWX by accessing [http://localhost/](http://localhost/) in your browser.

(Use `stop` to stop the container, and `destroy` to reset the containers and _all_ images.)

### Push the containers to Docker Hub

Currently, the process for updating this image on Docker Hub is manual. Eventually this will be automated via Travis CI using `ansible-container push` (currently, this is waiting on [this issue](https://github.com/ansible/ansible-container/issues/630) to be resolved).

  1. Log into Docker Hub on the command line:

         docker login --username=geerlingguy

  1. Tag the latest version (only if this is the latest/default version):

         docker tag awx_web:devel geerlingguy/awx_web:latest
         docker tag awx_web:devel geerlingguy/awx_web:1.x
         docker tag awx_task:devel geerlingguy/awx_task:latest
         docker tag awx_task:devel geerlingguy/awx_task:1.x

  1. Push tags to Docker Hub:

         docker push geerlingguy/awx_web:latest # (if this was just tagged)
         docker push geerlingguy/awx_web:1.x
         [etc...]

## License

MIT / BSD

## Author Information

This container build was created in 2017 by [Jeff Geerling](https://www.jeffgeerling.com/), author of [Ansible for DevOps](https://www.ansiblefordevops.com/).
