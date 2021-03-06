<!-- Docker Management -->

# <img src="../.pics/Lexxeous/lexx_headshot_clear.png" width="100px"/> Lexxeous's Docker Management: <img src="../.pics/Docker/docker_logo.png" width="150"/>

> Credit to freeCodeCamp for providing the [Docker Tutorial for Beginners - A Full DevOps Course on How to Run Applications in Containers](https://www.youtube.com/watch?v=fqMOX6JJhGo) YouTube video tutorial.

> For additional, hands-on practice with **Docker**, you can access free labs from KodeKloud [here](https://beta.kodekloud.com/lessons/hands-on-labs/).

## 1. Summary:

**Docker** is a containerization platform that allows the creation and isolation of software applications. Rather than having to worry about a plethora of dependencies and compatible operating systems for an application, **Docker** allows you to separate the components of your application into managable pieces. Take a website, for example. For a simple website you may need to use things such as `NodeJS` with hosting as your web server, `MongoDB` or `SQL` for your database, `Redis` for your messaging system, and `Ansible` for application orchestration. All of these different components may require different libraries, dependencies, operating systems, development/test/production environments, and hardware infastructure to run together properly. This can easily cause a complicated mess when trying to run all of these services as one large application.

**Docker** however allows interoperability of these components as microservices. Each of these components will become its own container, complete with its own set of internal dependencies, not dependent on one another or the underlying OS, but rather sharing an OS kernel. This means that any Linux container can run with any Linux **Docker** installation, Windows with Windows, & MacOS with MacOS. Furthermore, it is possible to run a Linux container on a Windows OS, for example, but under the hood, **Docker** creates a Linux VM that runs in coordination with the Windows OS and hardware. A container is not meant to host an internal OS and the container only lives as long as the process inside of it is alive.

## 2. Containerization vs. Virtualization:

<img src="../.pics/Docker/container_vs_vm.jpeg" width="600px"/>

The image above gives a graphical representation on how containerization differs from virtualization. Using a containerization platform like **Docker** allows for applications to have a much smaller footprint, as they can share the same OS kernel underneath. This is different from VMs in the fact that each VM requires its own guest OS to run on top of the host OS, creating larger and more complex applications. Using containerization instead of virtualization will automatically cut down on CPU utilization, boot up time, and necessary disk space for the same set of running applications.

<img src="../.pics/Docker/cont_and_vm.png" width="600px"/>

In larger enterprise environments, it is also common to use both containerization and virtualization together. This allows a hypervisor to manage different types of virtual machines with different host operating systems, while the containerization environment runs necessary applications on top of different shared OS kernels. These types of configurations can be very powerful.

## 3. To DevOps:

Traditionally, developers would create an application and a list of instructions on how to set up the application for the operators to deploy on some hardware. These instructions contained things like necessary software libraries, dependencies, network configurations, etc. When the operators struggled to properly deploy the application to production, they would have to backtrack and work with the developers to solve the issue, considering that the operators themselves did not develop the application. With **Docker**, the development process changes. Rather than sending an application and a list of instructions on how to depoy the application manually, **Docker** allows the use of images and `Dockerfile`s as a alternative. The **Docker** image is a self-contained entity that already provides all of the necessary libraries, dependencies, configurations, and other requirements needed to build/deploy an application on top of any hardware infastructure and OS. In this way, the developers and the operators are combined in to a team commonly known as *DevOps*, changing the landscape of application deployment. With **Docker**, building and deploying an application is as simple as aquiring and running the **Docker** image and in the desired location.

## 4. Commands:

> Any **Docker** CLI command can be sent and ran remotely on a separate host that has a **Docker** engine installed. The **Docker** engine is built in layers where the CLI communicates with the REST API and the REST API communicates with the **Docker** deamon. This layered structure interprets CLI commands to interact with the native **Docker** installation. You can send CLI calls to a remote **Docker** engine by using `docker -H=<host_name_or_ip>:<port> <command>`.

</hr>

#### 4.i. Run:
If the image does not exist locally, **Docker** will automatically pull the image from a remote location and use the local image for any subsequent runs.
```bash
$ docker run [OPTIONS] IMAGE[:TAG] [COMMAND] [ARG...] # run container from a Docker image

	# Use the `-d` (detach) option to run a container in the background.
	# Use the `-p <host_port>:<container_port>` (publish) option to bind a host port to an exposed container port.
		# Specify a prefixed IP address with `<IP>:<port>`.
		# Specify a range of ports with `<start_port>-<end_port>`; quantity of ports in range must be the same for both host and container.
		# Specify a protocol to use with `<port>/tcp` or `<port>/udp`.
	# Use the `-i` (interactive) option to map the host stdin to the container's stdin.
	# Use the `-t` (terminal) option to allocate a pseudo-TTY from the container to the host.
	# Use the `-v` (volume) option to map volume data with `<host_path>:<container_path>`.
	# Use the `-e` (env) option to input a list of environment variables with `<VAR_NAME>=<val>`.
	# Use the `--name=<cont_name>` option to assign a name to a running container ; useful for Compose and DNS.
	# Use the `--link <cont_name>:<host_service>` option to link container services together.
	# Use the `--network=<network>` option to connect a container to a specific network.
	# Use the optional `[:TAG]` to specify an image version (default is "latest").
```

</hr>

#### 4.ii. Push:
This command can push a **Docker** image to a public or private registry in a remote or local location.
```bash
$ docker push [OPTIONS] NAME[:TAG] # push image or repo to a remote registry
```


</hr>

#### 4.iii. Pull:
This command is automatically run with `$ docker run` if the image is not found locally. You can use this command on its own if you only want to retrieve the image without also creating a container instance.
```bash
$ docker pull [OPTIONS] NAME[:TAG|@DIGEST] # pull a Docker image from a repo or registry
```

</hr>

#### 4.iv. Build:
This command is used to build a containerized, **Docker**ized image from a `Dockerfile`. In many cases, the `PATH` will just be the current working directory or the location of the `Dockerfile`, which would be `.` or `pwd`. It is good practice to put a comment at the top of your `Dockerfile` to specify what the build command should be for each **Docker** project.
```bash
$ docker build [OPTIONS] PATH | URL | - # build a Docker image from a Dockerfile
	# Use the `-t` (tag) option to specify a project name path like `<username>/<proj_name>`.
	# Use the `-f` (file) option to specify the name of your Dockerfile.
		# By default, the path to the Dockerfile is already set to 'PATH/Dockerfile'.
```

</hr>

#### 4.v. History:
This command allows you to view the detail of how a specific **Docker** image was built.
```bash
$ docker history [OPTIONS] IMAGE # show the history of a Docker image
```

</hr>

#### 4.vi. Process Status:
Prints container information such as the container ID, the image name, the current internal process command, the creation date, the current status, the ports used, and the alias name.
```bash
$ docker ps [OPTIONS] # list containers

	# You can use the `-a` (all) option to list all containers (both running and stopped).
```

</hr>

#### 4.vii. Inspect:
This command can return much more details about a **Docker** object/container than `$ docker ps`. It returns all container information indexed in a `JSON` format.
```bash
$ docker inspect [OPTIONS] NAME|ID [NAME|ID...] # return low-level Docker object info in JSON format
```

</hr>

#### 4.viii. Logs:
If you don't have a container running in interactive mode with `-i`, attached to a pseudo-TTY with `-t`, or are not printing logs to an internal or external volume log file, you can use this command to fetch the container's `stdout`.
```bash
$ docker logs [OPTIONS] CONTAINER # get the stdout logs from a specific container
```

</hr>

#### 4.ix. Start:
This command allows you to start up an already existing **Docker** container after it has been stopped (after acquiring an exit status). A **Docker** image will have created a container instance with a specific, randomized alias name. Use this command to "boot up" a pre-existing instance rather than using `$ docker run`. Using `$ docker run` will simply create a brand new container instance from the **Docker** image.
```bash
$ docker start [OPTIONS] CONTAINER [CONTAINER...] # start one or more stopped containers
```

</hr>

#### 4.x. Stop:
To stop a specific container you can use either the container ID or the alias name. This command will stop a container from running. You can start up the container again (rather than creating another container instance from an image) by using `$ docker start [CONTAINER_ID | ALIAS_NAME]`.
```bash
$ docker stop [OPTIONS] CONTAINER [CONTAINER...] # stop a running Docker container instance
```

</hr>

#### 4.xi. Remove:
To remove a specific container you can use either the container ID or the alias name. This command will simply remove the container instance, not delete the **Docker** image used to create the associated container instance.
```bash
$ docker rm [OPTIONS] CONTAINER [CONTAINER...] # remove a specified Docker container instance
```

</hr>

#### 4.xii. Images:
Lists local **Docker** images that can be immediately used to create container instances.
```bash
$ docker images [OPTIONS] [REPOSITORY[:TAG]] # list available, local Docker images
```

</hr>

#### 4.xiii. Remove Image(s):
Removes one or more local **Docker** images. First, ensure that there are no container instances running off of the image(s) to be removed.
```bash
$ docker rmi [OPTIONS] IMAGE [IMAGE...] # remove one or more Docker images
```

</hr>

#### 4.xiv. Execute:
While a specific **Docker** container instance is running, you can pass commands to it. It will then run these commands as internal processes. When a container no longer has any internal processes to run, the container instance will exit/halt/stop. 
```bash
$ docker exec [OPTIONS] CONTAINER COMMAND [ARG...] # execute a command in a running container instance
```

## 5. Dockerfiles:

> The following steps represent a guide for the ordered, layered architecture required for `Dockerfile`s:

1. All "commands" inside of a `Dockerfile` follow a simple `INSTRUCTION ARGUMENT` format.
2. All `Dockerfile`s must start with a `FROM` instruction that specifies a base OS or another environment image as a foundation.
3. Install necessary dependencies/libraries/packages for your **Docker**ized application to run.
4. Copy any required source code (current directory or otherwise) to a usable destination in the container's file system.
5. Specify an internal command as the entry point for the container to use when it is run.

> The build layers of a `Dockerfile` are cached in the host's memory during the build process. If any such layer should fail during the process, you can fix the problem and re-run the same build command without **Docker** needing to start the entire build process over from the beginning. The same is true for updating steps in a `Dockerfile` build and for any other `Dockerfile`s that have any same steps as another. **Docker** can retrieve the build steps from cache, saving disk space and time. These benefits are provided automatically without additional effort.

```dockerfile
FROM #comment

ARG

RUN

COPY

EXPOSE

VOLUME

ADD

ENV

WORKDIR

# Entrypoint instructions are dynamic, command line arguments passed from the `$ docker run`
# command will be appended here. You can also use the `--entrypoint <string>` option with
# `$ docker run` to overwrite the default `ENTRYPOINT` for the image
ENTRYPOINT # command
ENTRYPOINT # ["command"]

# Command instructions are semi-dynamic, CMD arguments are replaced entirely, no matter what
# is hard-coded into the Dockerfile. 
CMD # command param1 ... paramN
CMD # ["command", "param1", ... "paramN"]
CMD # param1 ... paramN ; just param(s) for default value in tandem with `ENTRYPOINT`

# NOTE: If you want to set a "default" argument, when one is not present on the command line,
# you can use both `ENTRYPOINT` & `CMD` together. Put the command to be run as an argument for
# `ENTRYPOINT` and the default argument to potentially be replaced as the argument for `CMD`.

HEALTHCHECK

```

## 6. Docker Storage:

### 6.i. Volumes:

```bash
# Create a new custom Docker volume that will be stored on the host machine
$ docker volume create <vol_name>

# Run a container with a custom Docker volume mounted to an internal container path (volume-mount)
$ docker run -v <vol_name>:<internal_path> <image_name>
# OR
$ docker run --mount type=volume,source=<vol_name>,target=<internal_path> <image_name>

# Run a container with a pre-existing host path mounted to an internal container path (bind-mount)
$ docker run -v <host_path>:<internal_path> <image_name>
# OR
$ docker run --mount type=bind,source=<host_path>,target=<internal_path> <image_name>

# List the available custom Docker volumes
$ docker volume ls
```

### 6.ii. Image Layers vs. Container Layer

The following image gives an example of how **Docker**'s layered architecture works with regards to storage and data access:

<img src="../.pics/Docker/layered_arch.png" width="600px"/>

In this example, a `Dockerfile` exists that builds an image from a base Ubuntu OS, updates and installs `apt` libraries, installs `Python`/`pip` packages, manages the source code, & defines the entry point. These 5 instructions are contained within the image layers of the layered architecture. During the build process, **Docker** creates the image and these layers to be "read-only". All duplicate running containers will share the same "read-only" image layers. Upon running the image, **Docker** creates a container instance and a container layer. This layer's data access is "read-write" but only exists as long as the container is alive (not in an exited state). New files and folders that are created for the running container will then exist in the container layer. When the container reaches its exited state, all files and folders inside of the container layer are destroyed.

> It is possible to edit a file that exists in the image layer, but the file will first be copied to the container layer before editing. This is called "copy-on-write".

### 6.iii. Storage Drivers:

> The storage driver that is chosen to be used for a running container is automatically determined by **Docker** based on the operating system.

  * AUFS (Advanced multi-layered Unification File System) – Default for Ubuntu
  * ZFS (Zettabyte File System)
  * BTRFS (B-TRee File System)
  * Device Mapper
  * Overlay
  * Overlay2

## 7. Docker Networking:

  * An internal IP address is assigned to all containers when they are running. In most cases, this internal IP address is typically within the range of `172.17.0.Z`, starting from `Z = 2`. The IP address of `172.17.0.1` is used for the virtual bridge interface `docker0` created between the operating system and the **Docker** application.
  * For more detailed network information about a specific container, you can use `$ docker inspect <alias_name>`. You can find all of the network configuration information about the container under the `["NetworkSettings"]` index of the `JSON` output.
  * There are 4 kinds of networks available for **Docker** containers:

### 7.i. Bridge:

> This network is used by default. You do not need to use the `--network` option with `$ docker run` in this case.

With this network option, manual port mapping is used by default. The ports used on the host and the ports used for the running container(s) are not the same. Internal container ports must be exposed and host ports must be mapped to these exposed container ports. With this type of configuration, it is possible to run more than one container with the same internally exposed port from different external host ports.

### 7.ii. Null/None:

> This network is not used by default. To use this network you need to specify `--network=none` with the `$ docker run` command.

With this network option, running containers are not attached to any network. This means that they have no access/connection/communication with the external network or with any other containers. They run in a completely isolated fashion. These types of containers may be useful as independent compute nodes that simply write results to a file on an internal or external volume.

### 7.iii. Host:

> This network is not used by default. To use this network you need to specify `--network=host` with the `$ docker run` command.

With this network option, there is no manual port mapping necessary. All internal container port numbers are automatically mapped to the same host port number. With this type of configuration, it is NOT possible to run more than one container with the same internally exposed port, as this would attempt to map two or more containers to the same external host port.

### 7.iv. User-defined:

> This network is not used by default. With user-defined networks, we can separate the network groups of any running containers. To setup a user-defined network, you must use the `$ docker network create` command.

```bash
# Create new custom container network
$ docker network create --driver <driver_type> --subnet <W.X.Y.Z>/<mask> <cust_network_name>
	# The supported values for <driver_type> are bridge, none, and host.
	# Devs commonly create subnets following an incremented pattern with the default 172.17.0.Z scheme.
		# For example, user-defined network group IPs would be 182.18.0.Z, 192.19.0.Z, and so on.

# List available container networks
$ docker network ls
```

### 7.v. Embedded DNS:

**Docker** already has a built in `DNS` service that is used in the background and maps running container names to thier cooresponding IP addresses. The default IP address for this embedded **Docker** `DNS` service is `127.0.0.11`. The proper way to internally reference containers from other containers or from commands is to use its "name" rather than its internal IP address. It is not guaranteed that any container will obtain the same internal IP address any time that a container is rebooted.

## 8. Docker Compose:

> The following example is available as an official **Docker** demo repository [here](https://github.com/dockersamples/example-voting-app).

<img src="../.pics/Docker/compose_demo_flow.png" width="400px"/>

To start all 5 of these containers and connect them, we can use the following 5 commands:

```bash
$ docker run -d --name=redis redis
$ docker run -d --name=db postgres:9.4
$ docker run -d --name=vote -p 5000:80 --link redis:redis voting-app
$ docker run -d --name=result -p 5001:80 --link db:db result-app
$ docker run -d --name=worker --link redis:redis --link db:db worker
```

However, rather than using all of these `$ docker run` commands separately, we can use **Docker** Compose to create an IaC-type `.yml` file that will concatenate all of this information for us. The `docker-compose.yml` file below is equivalent to running all 5 of the above CLI commands, with the addition of some ordering and networking configurations.

> For the following `.yml` file, **Docker** Compose assumes that all of the images are already available. But the voting, result, and worker applications will be custom. You can instead pass a path to a build key inside of the `docker-compose.yml` file for each custom container like `build: <source_path>`. The source path for each image must contain all references to necessary application source code and the `Dockerfile` required to build the associated application image.

```yaml
# docker-compose.yml
version: 2

services:
	redis:
		image: redis
		networks:
			- back-end

	db:
		image: postgres:9.4
		networks:
			- back-end

	vote:
		image: voting-app
		ports:
			- 5000:80
		links:
			- redis
		depends_on:
			- redis
		networks:
			- front-end
			- back-end

	result:
		image: result-app
		ports:
			- 5001:80
		links:
			- db
		depends_on:
			- db
		networks:
			- front-end
			- back-end

	worker:
		image: worker
		links:
			- redis
			- db
		depends_on:
			- redis
			- db
		networks:
			- back-end

networks:
	front-end: # custom network setup/config
	back-end: # custom network setup/config
```

> The `depends_on` key support was added with **Docker** Compose version 2 and allows for the specification of start container start up ordering. For more information about **Docker** Compose file versioning, see the [documentation](https://docs.docker.com/compose/compose-file/compose-versioning/).

Finally, we can use the following command to start up a group of containers. The default path/filename for the **Docker** Compose file is `./docker-compose.yml`, but you can specify different `.yml` configuration file(s) with the `-f` option.

```bash
$ docker compose up
```

## 9. The **Docker** Registry:

The location of **Docker** images for the online registry follows a certain format:

`<website_name>/<user_account>/<image_name>:[version_tag]`

The defaults for pulling remote images are as follows:
  
  * If you don't specify the website name, the default is `hub.docker.com`.
  * If you don't specify the user account name, the default is the same as `<image_name>` (primarily for offical **Docker** images).
  * If you don't specify the version tag, the default is `latest`.

### 9.i. Private Registries:

No content.

## 10. Control Groups ("cgroups"):

By default, all containers have no restrictions on the hardware resources they utilize from the host machine. This means that without specifying restrictions, a collection of containers could potentially use 100% of the host machine's CPU and memory. Although this may be desired in some cases, we can avoid this by denoting certain limits prior to running a container. While using the `$ docker run` command we can use a variety of options to control container resource utilization:

  * `--cpus <decimal>` – Specify the ratio of CPU usage allowed.
  * `--memory <bytes>` – Specify the maximum memory usage allowed in bytes.
  * `--device-read-iops <list>` – Specify the maximum read I/O operations per second for connected devices.
  * `--device-write-iops <list>` – Specify the maximum write I/O operations per second for connected devices.
  * `--device-read-bps <list>` – Specify the maximum data read bits per second rate for connected devices.
  * `--device-write-bps <list>` – Specify the maximum data write bits per second rate for connected devices.
  * `--pids-limit <int>` – Specify the maximum number of live processes allowed for the container's PID namespace.

> For more information about **Docker** container runtime options and resource constraints, consult the [documentation](https://docs.docker.com/config/containers/resource_constraints/).

## 11. Misc:

  * By default, **Docker** containers run in an uninteractive mode and do not listen to `stdin`.
    - Use `-i` with `$ docker run` to run a **Docker** container in interactive mode.
    - Use `-t` with `$ docker run` to attach the **Docker** container terminal's `stdout` to the host's.
  * With `$ docker inspect` you can find all of the environment variables that a particular running container is using under the `["Config"]["Env"]` index of the `JSON` output.
  * Container orchestration can be achieved with:
    - **Docker** Swarm
    - Google's **Kubernetes**
    - Apache **Mesos**




