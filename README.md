# Rundeck docker

## About
This project maintains a Rundeck docker.

## Build
To build the docker image manually, check out this repository and run

`docker build -t <image-name> .`

## Run
There are two required environment variables for this Rundeck to run. So to use any
run command below, please add `-e` flag to set `SSH_IP` and `SSH_USER` variables.

`docker run -e SSH_IP="<remote_ip>" -e SSH_USER="<remote_user>"`

More details can be found at section [Run remote jobs from Rundeck](#run-remote-jobs-from-rundeck).  
 
This Rundeck docker exposes two ports 4440 and 4443 for HTTP and HTTPS respectively.
If you build the docker with image name 'platform-rundeck', then you can map
docker's internal port 4440 to your localhost with command:

`docker run -d -p 4440:4440 platform-rundeck`

`-d` flag is used to run the docker in daemon mode.

## Custom configuration
To configure rundeck, please refer to [RPM layout](http://rundeck.org/docs/administration/configuration-file-reference.html#rpm-layout).
If you would like to override any of the default configurations, please provide
a complete configuration file and mount it at `/mnt/config` usig the -v flag when
running the docker.

For example, if you want to override realm.properties, then create a file
realm.properties on your host. Suppose that your file resides at
/home/rundeckConfigs, then you can mount it as:

`docker run -v /home/rundeckConfigs/:/mnt/config/ -p 4440:4440 platform-rundeck`

If the mounted directory (/home/rundeckConfigs in this example) contains any
configuration files recognized by rundeck, they will override the defaults provided
by rundeck.

## Rundeck Plugins
[Plugins](http://rundeck.org/plugins/) supported by Rundeck can be added my mounting
the plugin jar files on path /mnt/plugin in the docker. For example, if
your plugins are located at /home/rundeckPlugins on your host machine, you can run:

`docker run -d -v /home/rundeckPlugins/:/mnt/plugin/ -p 4440:4440 platform-rundeck`

Loaded plugins can be viewed from Rundeck web UI.

## Rundeck Resource Model Source
Rundeck provides a variety of ways to add [Node Sources](http://rundeck.org/docs/administration/managing-node-sources.html).
You can mount node definition files on path /mnt/resource in the docker. For example,
if your resource files are located at /home/rundeckResources on your host machine,
you can run:

`docker run -d -v /home/rundeckResources/:/mnt/resource/ -p 4440:4440 platform-rundeck`

You can now add the resources through the Rundeck web UI, just point the location
to directory /mnt/resource.

## Run remote jobs from Rundeck
Jobs on remote machine can be run using ssh. By default, the current rundeck docker 
will set up one remote node. For Linux machine, your host machine will be set as the 
remote node. For Mac/Windows, your local docker machine will be set as the remote node. 
The required environment variables are SSH_IP and SSH_USER and the required data volumes
are ~/.ssh/ for Linux and ~/.docker/machine/machines/default/ for Mac/Windows.

Linux only: before starting the Rundeck docker, make sure openssh-server is installed and up running.
You can install and start openssh server by the following commands:
`
apt-get install openssh-server
service ssh start
` 

Linux user can run the rundeck docker using:
```
  docker run -d -v ~/.ssh/:/mnt/ssh \
   -e SSH_USER=<USERNAME_ON_YOUR_MACHINE> \
   -e SSH_IP=<YOUR_MACHINE_IP_> \
   -p 4440:4440 \
   platform-rundeck
```

Mac/Windows user can run the rundeck docker using:
```
  docker run -d \
   -v ~/.docker/machine/machines/default/:/mnt/docker/ssh \
   -e SSH_USER=docker \
   -e SSH_IP=192.168.99.100 \
   -p 4440:4440 \
   platform-rundeck
```
You can set up other remote nodes by the following steps after starting the rundeck docker.

If the remote machine is configured as a resource (via resource
  model definition), then you only need to perform step 1.

1. Copy the public ssh key from rundeck docker to the remote machine's authorized_keys
file. If the rundeck docker is running on the remote machine, you can use the following
`docker cp` command to accomplish this:

`docker cp <rundeck_container_id>:/var/lib/rundeck/.ssh/id_rsa.pub ~/.ssh/authorized_keys`

Location of authorized_keys file may vary.

2. You may need to use the option `-oStrictHostKeyChecking=no` with the ssh command
in the rundeck docker. This flag is only required if the remote machine has not been
verified as a valid host before. For simplicity, use the following command to add
a remote machine to known_hosts file in the rundeck docker:

`docker exec <rundeck_container_id> sudo -u rundeck ssh -oStrictHostKeyChecking=no -i /var/lib/rundeck/.ssh/id_rsa <user@ipAddress> 'exit'`

Replace user@ipAddress with username and ip address of the remote machine.

3. Now you can run a rundeck job on a remote machine by using ssh. For example, the
following job command will execute a 'docker ps' job on a remote machine with ip
address 192.168.66.2:

`ssh user@192.168.66.2 'docker ps'`
