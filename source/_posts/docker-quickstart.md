---
title: Docker Quickstart
date: 2019-06-07
tags: [docker]
---

Docker seems to be harder to setup than it should be. In this post i'm going to share a quick script to get you setup right.

```bash
#!/usr/bin/env bash

command -v curl >/dev/null 2>&1 || { echo "I require curl but it's not installed." >&2; }

# Variable declarations
COMPOSEVERSION="1.24.0"

# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
docker -v

#Add current user to docker group
sudo usermod -aG docker $USER

#Install Docker-compose
sudo curl -L "https://github.com/docker/compose/releases/download/$COMPOSEVERSION/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version

# Bash convenience scripts
sudo curl -L https://raw.githubusercontent.com/docker/docker-ce/v$(docker -v | cut -d' ' -f3 | tr -d '\-ce,')/components/cli/contrib/completion/bash/docker -o /etc/bash_completion.d/docker
sudo curl -L https://raw.githubusercontent.com/docker/compose/$(docker-compose version --short)/contrib/completion/bash/docker-compose -o /etc/bash_completion.d/docker-compose

# Create daemon.json
#docker can also create its own user for usernamespaces with "userns-remap": "default"
sudo cat << EOF > /etc/docker/daemon.json
{
  "storage-driver": "overlay2",
  "graph": "/var/lib/docker",
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "25m",
    "max-file": "2"
  },
  "debug": false,
  "userns-remap": "$(id -u):$(id -g)"
} 
EOF

# Restart docker
sudo service docker restart

echo "Logout and log back in for user to be added to docker group"

```