# Rπ home server

Simple tool for provisioning my home Raspberry Pi.

## What?

I have a Raspberry Pi as a little home server(who doesn't?) and i come up with this simple AF solution for put everything back after i replaced the SD card.

**It's this k3s?** No.
**It's this PiCluster?** Nop, sorry.
**It's this Balena?** No no, stop it.

I'm a simple man with simple needs, i run a bunch of docker containers in my Pi with services that i think are nice to have at home(like pihole). It doesn't have K8S or anything like that...for now.
This is basically an Ansible Playbook that:

  1. Setups the pi: configures locales, timezone, keyboard, sets an static ip, connects to Wi-Fi, upgrades the system and install packages.
  2. Installs Docker & docker-compose.
  3. Copies docker-compose projects to the Pi.
  4. Copies your backed up files of the docker volumes to the Pi.
  5. Starts all containers.

For the setup part i use a personal role that you can check out [here](https://github.com/acimadamore/ansible-role-raspberry-pi). Docker, on the other hand, is installed using the amazing _docker_arm_ role from geerlingguy's.

## i've give it a shot, how do i use it?

Take your Raspberry Pi with a fresh install of Raspbian OS and enable SSH using _raspi-config_ or placing an empty file named _"ssh"_ on the boot partition before you start it up. Copy your SSH key to your pi:

> $ ssh-copy-id -i <path-to-your-ssh-key> pi@<your-pi-ip-address>

Install Ansible and other required packages on your computer:

> $ pip3 install -r requirements.txt

Install the required roles:

> $ ansible-galaxy install -r requirements.yml

In the _inventory_ file put yours Pi current IP address. In _var/raspberry_pi.yml_ put your desired configuration for the basic setup(see the available variables [here](https://github.com/acimadamore/ansible-role-raspberry-pi)).
In the _docker/compose-projects_ folder place all the docker-compose and other files that you make your server, for example:

```
docker/compose-projects/
        pihole/
           docker-compose.yml
        portainer/
           docker-compose.yml
        my-personal-site/
           docker-compose.yml
           Dockerfile
           index.html
           css/...
           js/...
```

All this files will be copied as they are to your Pi. Finally if you wish to restore yours Docker volumes' content, copy all the files to _docker/volumes_ inside a folder named after the volume, so if you want to restore a volume named _"pihole_conf"_ the files should be at _docker/volumes/pihole_conf_.

After everything is in place run your playbook with:

> ansible-playbook playbook.yml -i inventory -K

Sit back and relax.

## Configurations and gotchas

If you know what are you doing you can change basically everything of this playbook. There are a couple of variables in the _vars_ section though that maybe are usefull:

- **docker_compose_files_path** (/home/pi/services): the path where the compose-projects files will be copied.

- **copy_volumes**(true): if you want or not to restore volumes.

- **always_replace_docker_compose_projects**(true) and **always_replace_docker_volumes_files**(true): whether or not to replace the files already in the Pi when you copy your compose-projects and volumes respectively.

This last two options are useful if you want to keep managing your RPi using this playbook. You could add new docker-compose files, change some settings on a volume file, rerun the play and the changes will apply to the Pi, but beware, if some of this files are changed by the containers(specially those of the volumes) this options will discard them.

Finally two assumptions are made: every docker-compose project in _docker/docker-projects_ has its own folder and there's only one YAML file in the root of these folders acting as the docker-compose.yml file and finally is assumed that you'll be running the playbook as the default _pi_ user and use sudo for privilege scalation, but that can be changed in the playbook.
