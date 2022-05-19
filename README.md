# Docker Server Project
This repository contains a full media-entertaiment application stack running in my headless Linux server (Ubuntu server 22.04 LTS).
I will be deploying 4 containers for this project.

After searching for the perfect NAS solution, I realized what I wanted could be achieved with some Docker containers on a vanilla Linux box. The result is an opinionated Docker Compose configuration capable of browsing indexers to retrieve media resources.
## Hardware Configuration

- [x] **Router**

[NETGEAR R700](https://www.amazon.com/NETGEAR-Nighthawk-Smart-Wi-Fi-Router/dp/B00F0DD0I6) running on [DD-WRT v3.0-r48432 std](https://wiki.dd-wrt.com/wiki/index.php/Netgear_R7000)

- [x] **Server**
![This Is My Server](images/server.jpg)

This is basically an excellent Dell Optiplex 7050 I got off Ebay for cheap and made a couple of changes.
- Ubuntu 22.04 LTS
- Intel I7-6700 CPU
- Intel HD Graphics 530 {I will be switching it for a dedicated GPU, mainly for Tanscoding}
- Killisre DDR4 2x8GB running at 2133Mhz 
- Low profile AMD Radeon HD 8570 (Completely useless, Not in Use)
- Kingston RBU-SNS8100S3 128GB M.2 SATA (for the OS and all the containers files and databases)
- [Seagate BarraCuda 8TB](https://www.amazon.com/Seagate-External-Desktop-Photography-STEL8000100/dp/B01HD6ZLQ6?th=1) (Yes, it's an SMR drive, but I got it cheap and shucked it. I think It is decent as a media drive as there won't be much writing involved)

![Ubuntu Server](images/ipserver.jpg)

## Installation

I will be using docker-compose for this setup. Each container's files and databases will be installed in its own directory on my home directory. I decided to create a separate .yml file for each container, although one could be created for all the containers. 
My home directory looks like this:

```console
/home/op/docker-compose
├── plex
│   ├── config
│   └── docker-compose.yml
├── sonarr
│   ├── config
│   └── docker-compose.yml
├── nzbget
│   ├── config
│   └── docker-compose.yml
├── radarr
│   ├── config
│   └── docker-compose.yml
└── portainer
    ├── config
    └── docker-compose.yml

```
![This Is my Home Directory](images/folders.jpg)


**Installing Docker and Docker-compose

**- Why Docker?**

Normally to deploy an application, you need to have an OS set up and configured, adding repositories, getting the app;ications, installing all the pre-requisities and dependencies. Each of these can be avioded using docker as it comes with everything already packed inside each container, like a mini OS. With `docker-compose` , you can edit and configure each container's configuration and parameters. I will be using it.

**- Setting up the repository**

```console
$ sudo apt update
$ sudo sudo apt install apt-transport-https ca-certificates curl software-properties-common gnupg lsb-release
$ echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] {{ download-url-base }} \
      $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
$ sudo apt update
```

**- Installing Docker Engine**

```console
$ sudo apt-get update
$ sudo apt-get install docker-ce 
```

**- Installing Docker-compose**

```console
$ sudo curl -L https://github.com/docker/compose/releases/download/v2.5.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose
 ```
 -Add Linux User to Docker Group
 
 So that you don't have to sudo every time. 
 Replace `op` by your user, type `whoami` to get it if you happen to do not know. 

```console
$ sudo usermod -aG docker op
```

**- Mounting Media Disk

This is where all the media files will downloaded, stored and accessed by Plex

Lets first identify the parttion you want to mount

```console                   
$ lsblk                   
```
 The ouptpout should be something like this 


```console 
NAME    MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sdb       8:32   0   7.3T  0 disk   
└─sdb1    8:33   0   7.3T  0 part
 ```
Mine is `sdb1`

Now, check the filesystem type of the disk or partition.

```console 
$ blkid /dev/sdb1
 ```
Output
```console 
/dev/sdb1: UUID="3a78sf00-a278-45aa-9260-4a98f9f8f8cf" TYPE="ext4" PARTUUID="cb5r8dba-5v0d-48c6-a95e-c89c4521c8bf" 
 ```
 Now you can create a directory for mount point and manually mount your desired partition. I will just mount mine on `/mnt/disk/`
 
```console 
$ sudo mount -t ext4 /dev/sdb1 /mnt/disk
 ```
 It should now be mounted. Go ahead and type `df -h` to confirm it was successfully mounted.
 
 To automatically mount your partition at every reboot, you will have to add an entry for a new mount point in /etc/fstab
 
 ```console 
$ sudo vi /etc/fstab/
 ```
 add this line (change the UUID with your own disk partition) using any editor
 `UUID="3a78sf00-a278-45aa-9260-4a98f9f8f8cf /mnt/disk/   ext4    defaults        0       0`




## Setup Environmental Variables for Docker-compose

Look back up at my the folder structure I mentioned earlier Into each container's folder, create a docker-compose.yml file. This is how myfolder structure looks like
 
```console
/home/op/docker-compose
├── plex
│   ├── config
│   └── docker-compose.yml
├── sonarr
│   ├── config
│   └── docker-compose.yml
├── nzbget
│   ├── config
│   └── docker-compose.yml
├── radarr
│   ├── config
│   └── docker-compose.yml
└── portainer
    ├── config
    └── docker-compose.yml
```

**Plex**

```console
$ touch docker-compose.yml
$ nano docker-compose.yml
```
Now, edit the .yml file 

```console
---
version: "2"
services:
  plex:
    image: linuxserver/plex
    container_name: plex
    network_mode: host
    environment:
      - PUID=PUID:-1000
      - PGID=PGID:-1000
      - VERSION=docker
      - TZ=America_New_York
    volumes:
      - /home/op/docker-compose/plex/config:/config
      - /mnt/disk/media/:/media/
    restart: unless-stopped
```

Note that your PUID and PGUID can both be retrieved using the command `id user` . Replace user by your user account name.


**Sonarr**

```console
---
services:
  sonarr:
    image: linuxserver/sonarr
    container_name: sonarr
    network_mode: bridge
    environment:
      - PUID=PUID:-1000
      - PGID=PGID:-1000
      - VERSION=docker
      - TZ=America_New_York
    volumes:
      - /home/op/docker-compose/sonarr/config:/config
      - /mnt/disk:/mnt/disk
    ports:
      - 8989:8989
    restart: unless-stopped
```

**Radarr**

```console
---
version: "2"
services:
  radarr:
    image: linuxserver/radarr
    container_name: radarr
    network_mode: bridge
    environment:
      - PUID=PUID:-1000
      - PGID=PGID:-1000
      - VERSION=docker
      - TZ=America_New_York
    volumes:
      - /home/op/docker-compose/radarr/config:/config
      - /mnt/disk/:/mnt/disk
    ports:
      - 7878:7878
    restart: unless-stopped
```
**Nzbget**

```console
version: "2"
services:
  nzbget:
    container_name: nzbget
    image: ghcr.io/hotio/nzbget:latest
    restart: unless-stopped
    logging:
      driver: json-file
    network_mode: bridge
    ports:
      - 6789:6789/tcp
    environment:
      - PUID=1000 
      - PGID=1000 
      - TZ=America_New_York
    volumes:
      - /home/op/docker-compose/plex/config:/config
      - /mnt/diskk/usenet/adult:/mnt/disk/usenet/
    restart: unless-stopped 
```


**Portainer**

If you happen to have many containers, or do not feel confortable working with CLI, `Portainer` can help you manage your stack through a Web UI. 

---
version: "2"
services:
  portainer:

       image: portainer/portainer-ce:latest
       container_name: portainer
       restart: unless-stopped
       security_opt:
                    - no-new-privileges:true
       volumes:
                - /etc/localtime:/etc/localtime:ro
                - /var/run/docker.sock:/var/run/docker.sock:ro
                - ./portainer-data:/data
        ports:
             - 9000:9000
```


After saving all the docker-compose.yml files, run the following command from each container directory to pull and start the container:

```console
$ docker-compose up -d

![This Is My Portainer Main](images/portainer.jpg)```

Now, all you have to do is open your browser to access your application `http://your_server_ip:port`
To access my plex server, I would enter 

```console
http://10.1.1.10:32400
```

For more information on how to configure Plex server's settings, Sonarr's settings, and all the file permissions, please follow these guides:
- [WikiArr](https://wiki.servarr.com/docker-guide)
- [Trash-Guides](https://trash-guides.info/How-to-setup-for/Docker/)


