---
title: " Hosting a CTF "
date: 2021-09-04T15:40:28+05:30
author: "Somesh Banerjee"
authorTwitter: "banerjee_somesh"
cover: ""
tags: ["InfoSec", "CTF"]
keywords: ["Linux", "CTF", "CTFd"]
description: "How I Hosted a Ctf for my School Club using a VM instance and CTFd repository"
readingTime: true
draft: false
---

# Introduction

Creating a full-fledged platform to host a CTF is very time taking. Moreover, this task looks almost impossible if you are not a developer. This blog will tell how I created the CTF platform for my School InfoSec club using CTFd within a few minutes.

# Choosing the Server


To host the platform, you need a server that will be accessible to everyone. For the server, you can create one from Digital Ocean. You have to decide what specifications you require based on the expected participants. In my case, I expected 100 participants, so I selected the following specifications:
- RAM: 8GB
- Cpu: 4vCPUs
- Storage: 100GB
- Version: Ubuntu 20.04 LTS x64

After the instance is created, Digital Ocean will provide you with an IP address and password. You can log in to your server with the credentials using Putty

# Setting up CTF Platform

1. After logging in to the server, the first thing you need to do is update it and install the requirements. You will only need to install git, docker, and docker-compose.
    ```bash
    sudo apt update
    sudo apt install docker docker-compose git
    ```

2. Now we need to clone the [CTFd](https://github.com/CTFd/CTFd) repo and move into it.
    ```bash
    git clone https://github.com/CTFd/CTFd
    cd CTFd
    ```

3. Create a random key. This is used by the Application for security purpose. I am storing the key in the `.ctfd_secret_key` file.
    ```bash
    head -c 64 /dev/urandom > .ctfd_secret_key
    ```

4. (Optional) Change the number of workers in `docker-compose.yml` file. The higher the number the better it can manage traffic. In my case, I used 4 workers.

5. Running the platform
    ```bash
    sudo docker-compose up
    ```

That's it, now your platform will be running in port 80 and 8080. You can test it from your machine by going to `localhost:8080` or using any other machine and going to the given IP. 

Now you need the rest of the setup from the platform. The guide for that setup is available at the Repo.

The basic requirement for the platform is covered. But if you are hosting it at a global level, then you need to implement security feature like Rate limiting and Firewall to the server. You also need the get a domain for your IP from GoDaddy or somewhere else. You can check this [article](https://medium.com/csictf/self-hosting-a-ctf-platform-ctfd-90f3f1611587) for more. 

# Firewall Setup

I will cover the basic steps required to setup a firewall which will only allow http/https and SSH traffic to the server.

```bash
sudo apt update
sudo ufw allow 'Nginx Full'
sudo ufw allow 'OpenSSH'
sudo ufw enable
```
