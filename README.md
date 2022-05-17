# Docker Media-Server Project
This repository contains a full media-entertaiment application stack running in my headless Linux server (Ubuntu server 22.04 LTS).
I will be deploying 4 conatiners for this project.

After searching for the perfect NAS solution, I realized what I wanted could be achieved with some Docker containers on a vanilla Linux box. The result is an opinionated Docker Compose configuration capable of browsing indexers to retrieve media resources and downloading them through a Wireguard VPN with port forwarding

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

## Installation

I will be using docker-compose for this setup. Each container's files and databases will be installed in its own directory on my home directory. I decided to create a separate .yml file for each container, although one could be create for all the containers. 
My home directory is set up this way

 /home/op/docker-compose/ = Home directory
  
  Plex
  =>>/docker-compose/plex/config
  =>>/docker-compose/plex/docker-compose.yml
  
  Samba
  
  =>>/docker-compose/samba/config
  
  =>>/docker-compose/samba/docker-compose.yml
  
  Radarr
  
  =>>/docker-compose/radarr/config
  
  =>>/docker-compose/radarr/docker-compose.yml

**Installing Docker and Docker-compose

├── plex

│      ├── config

│      ├── movies

│      └── tv

├── Samba
│   ├── config
│   ├── downloads
│   └── music

├── nzbget
│   ├── config
│   └── downloads
├── 
│   ├── config
│   ├── downloads
│   └── movies
└── sonarr
    ├── config
    ├── downloads
    └── tv
