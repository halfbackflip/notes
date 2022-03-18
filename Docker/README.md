# Docker 

## Definition
- High level tool for setting up containers
- Package applications to run anywhere and in any environments
- Isolated environments which share the kernel of an operating system

## OS Kernel
- Low level operating code that interacts with the hardware.
- Common linux Kernel across all Linux version.
- Windows has a different Kernel.
- Cannot run Windows and Linux containers on the same OS, i.e. a Windows based-container cannot be run on Linux.

## Containers vs VMs
- VMs require a hypervisor, which adds extra overhead and more isolation from the OS.
- Docker containers run on top of the OS Kernel, less overhead and more connection to the host OS.
- Containers can also be deployed on a VM...

## Docker Repository
- Public repo of public images
- Pre-packaged container images
- An image is a template used to create containers
- Containers are running instances
- Able to create your own images for customization, allows easy deployments

## Docker editions
- Community vs Enterprise
- Community is free
- Available on MAC, Windows, Linux, Cloud, etc.
- Enterprise has additional business support and additional features

## Environment Variables
- Move variable out of the environment and into docker env variables
- This allows you to use the docker run command to assign values to variables
- The docker inspect command will allow you to check individual environment variables

## Dockerfile
- Defines the container image
- Specify steps during the image build process (i.e. docker build)
- Requires a from command at the start to specify the OS version, i.e. "FROM UBUNTU"
- Allows you to run commands and copy files

## Docker Networking
- When you start docker, three networks are created by default:
- Users can define their own networking within the host too.

| **Network** | **Description** |
| --------------|-------------------|
|`Bridge`| Default network a container is attached to. Private internal network created on the host. Containers can access each other access outside requires port mapping.|
|`none`| Containers are not attached to any network and are completely isolated.|
|`host`| Associates the container directly to the host's network.|



## Embedded DNS
- Containers can reach other using either internal IPs or the container name
- Using a container hostname is the preferred method over connecting via the IP address

## Docker Storage
- Docker creates a file system under: /var/lib/docker
- When you deploy a container from an image for the second time onwards, the data is cached. 
- The new container re-uses this cached data, saving storage space.
- Layered deployment: 
    - Container Layer: Individual container storage
    - Image Layer: Cached data storage
- Volumes: Add a persistent volume. Creates a folder on the host that is mounted in the container. Persists data after deletion.
- Storage driver types: AUFS, ZFS, BTRFS, Device Mapper, Overlay, Overlay 2
- The storage driver type depends on the underlying operating system

| **Location** | **Description** |
| --------------|-------------------|
|`/var/lib/docker`|
|`/var/lib/docker/volumes`|volume storage location|
|`/var/lib/docker/volumes`|storage location|

## Docker Compose
- Use Case: When you have to run multiple containers, i.e. mongodb, ansible, redis, etc. Instead of running all the containers with "run", declare the infraastructure in yaml.
- The voting application - the classic demo application in docker. 
- [https://github.com/dockersamples/example-voting-app](https://github.com/dockersamples/example-voting-app)
- Docker compose versions: Docker evolved over time. Newer versions have more features and parameters in them.
- In docker compose file version 2 and up, you must specify the version you are running
- Docker compose allows you to create more complicated networks, i.e. a front-end or back-end network.
- This creates a 3-tier web application

## Docker Registry
- Central repository for docker images
- Naming convention: <registry>/<useraccount>/<repo>, i.e. docker.io/nginx/nginx/
- Docker and google provide public registries
- Cloud services often provide private registries for their repos.
- Private registries require logging in before pulling or pushing to the repo
- You can create your own local registry hosted in a docker registry container

## Docker Engine
- The docker engine is composed of three components:
 1. Docker CLI
 2. REST API
 3. Docker Daemon
- The docker cli can also be run on a remote port, i.e "docker -H=remote-docker-engine:2375"
- Namespace - PID
    - When Linux boots, its begins with a PID 1 then boots up numerous PIDs in order after this...
    - Containers are the same, they being with a PID 1 then boot numerous processes after this.
    - Exterior processes are then mapped to interior processes
- Control Groups:
    - Restricts the amount of resources, i.e. memory, cpu, storage, a container can utilise

## Docker on Windows
- Two options for install:
    1. Create a linux VM, then run containers inside the VM
    - Docker toolbox, an executable with a series of tools for setting up a vm in windows
    2. Docker Desktop for Windows - Uses microsoft's native Hyper-V virtualisation settings
- VirtualBox and Hyper-V cannot co-exist on Windows

## Docker on MAC
- Two options on mac:
    1. Docker Toolbox (VirtualBox)
    2. Docker Desktop for Mac; leverages Hyperkit for Mac

## Container Orchestration
- Why orchestrate?
- Allows you to scale applications automatically to meet demand
- Scale instances if a failure occurs
- A Container orchestration setup will have clusters of multiple hosts with multiple containers on each for redundancy
- Load balancing and sharing storage across hosts is possible
- Docker Swarm, Kubernetes, Mesos are options for container orchestration
- Kubernetes is supported across all public cloud platforms

## Docker Swarm
- Create clusters of hosts and distribute your containers
- 1x Host is a 'Swarm Manager' other hosts are 'Workers'
- Commands are run on the Swarm Manager, which are then copied to the Worker Nodes
- Create replicas of worker nodes

## Kubernetes
- Allows for massive scaling 
- Autoscaling options based on demand
- Rolling upgrades for containers and images
- Test upgrades using A/B methods
- Several network and storage brands are supported via plugins
- Kubernetes supports alternatives to docker container
- Architecture consists of clusters of worker nodes
- Master Node - has a control plane installed. Manages the worker nodes.
- kubectl - kubernetes cli used to deploy / manage kubernetes clusters

| **Command** | **Description** |
| --------------|-------------------|
|`FROM UBUNTU`| Lists all the current containers running AND stopped their id, image, when created|
|`RUN apt-get update`| Run the apt-get update commmand |
|`ENTRYPOINT <blah>`| Specify the very first command to run|
|`COPY <blah>`| Copy given code|
|`CMD ["bash"]`| Specifies the service command which will run on startup. In this case, bash.|
|`CMD ["sleep","5"]`| Specifies the service command which will run on startup. In this case, sleep for 5 seconds.|

## Containers
| **Command** | **Description** |
| --------------|-------------------|
|`docker run ansible`| runs an ansible image in a container. If the image is not present on the host, it will retrieve from docker hub |
|`docker run -d --name webapp nginx:1.14-alpine`|Runs an nginx container and names it webapp|
|`docker run Ubuntu--network=host`|Specify the network type to attach at runtime|
|`docker run -d ubuntu sleep 1000`|Runs an instance of the ubuntu image along with the command sleep 1000 at startup.|
|`docker ps`| Lists all the current containers running, their id, image, when created
|`docker ps -a`| Lists all the current containers running AND stopped their id, image, when created
|`docker stop ansible`| Stop the ansible container |
|`docker rm ansible`| Removes the ansible container | 
|`docker images`| Lists avilable images and their sizes | 
|`docker rmi splunk`| Remove the splunk docker image | 
|`docker pull splunk`| Pulls (downloads) the image from docker hub. Does not run it.  | 
|`docker push <image name>`| Makes your image available on the public docker registry.  | 
|`docker exec ubuntu_container cat /etc/hosts`| Executes a command on a docker container. Reads the hosts file on a container called 'ubuntu_container'|
|`curl -fsSL https://get.docker.com -o get-docker.sh`| downloads the docker easy install script |
|`sudo sh get-docker.sh`| runs the docker easy install script |
|`docker run -d ansible`| runs an ansible image in detached mode a container. |
|`docker attach dt234d`| Re-connects to a docker with an image id. |
|`docker rm -f $(docker ps -a -q)`|Deletes all docker containers|
|`docker rmi ubuntu`|Deletes the ubuntu image|
|`docker rmi -f $(docker images -aq)`|Deletes all docker images on a host|
|`docker redis:4.0`|Specifies a version tag. Runs the 4.0 version of redis.|
|`docker run -i <blah>`|Runs docker with interactive mode|
|`docker run -it <blah>`|Runs docker with interactive mode and a terminal|
|`docker run -p 80:5000 webimages/webapp`|Runs the image webapp with a port mapping of 80:5000|
|`docker run -v /opt/mapdir:/var/lib/mysql mysql-app`| The -v options maps the docker /mysql to /opt/mapdir in the mysql-app for data persistence after the app is stopped.|
|`docker inspect <container_id>`|Displays detailed stats for a given container app|
|`docker logs <container_id>`|Displays all the logs for a given container|
|`docker build -t webapp .`|Build an image named webapp in a cwd|
|`docker run -e GUI_COLOR=blue web-app`|Run a webapp and set the gui color variable to blue|
|`docker network create --driver bridge --subnet 192.168.1.0/24 custom-network`|Create a bridged network called custom-network|
|`docker network ls`|List all created networks|
|`docker --mount type=bind,source=/data/mysql,target=/var/lib/mysql mysql`|Mounts a volume called mysql in docker. Note the source/target format|
|`docker run -d --name <name> --link <target>`|Link a container to an existing container|
|`docker compose-build`||
|`docker compose-up`||



## Use Cases
- Docker simplifies the deployment of mutiple services, i.e. database, messaging, ngnix, different libraries are required, i.e. Matrix from Hell

docker run -d --name=clickcounter --line redis:redis -p 8085:5000 kodekloud/click-counter
unknown flag: --line