# docker-configuration

The repository aims to help docker engine installation on Linux / Ubuntu OS.

[[_TOC_]]

## Process

Source : https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository

### Uninstall old versions

```bash
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

### Install using the `apt` repository

Set up Docker's apt repository:
```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

Install the Docker packages:

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Manage Docker as a non-root user:

```bash
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
```

### Test

```bash
docker run hello-world
```

⚠️ If you initially ran Docker CLI commands using sudo before adding your user to the docker group, you may see the following error:

```bash
WARNING: Error loading config file: /home/user/.docker/config.json -
stat /home/user/.docker/config.json: permission denied
```

This error indicates that the permission settings for the `~/.docker/` directory are incorrect, due to having used the sudo command earlier.  
To fix this problem, either remove the `~/.docker/` directory (it's recreated automatically, but any custom settings are lost), or change its ownership and permissions using the following commands:

```bash
sudo chown "$USER":"$USER" /home/"$USER"/.docker -R
sudo chmod g+rwx "$HOME/.docker" -R
```

### Configure Docker to start on boot with systemd:

```bash
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```

## Logs management

Source: https://docs.docker.com/engine/logging/drivers/json-file/

By default, Docker captures the standard output (and standard error) of all your containers, and writes them in files using the JSON format. The JSON format annotates each line with its origin (stdout or stderr) and its timestamp. Each log file contains information about only one container.

```json
{
  "log": "Log line is here\n",
  "stream": "stdout",
  "time": "2019-01-01T11:11:11.111111111Z"
}
```

To use the `json-file` driver as the default logging driver, set the `log-driver` and `log-opts` keys to appropriate values in the `daemon.json` file, which is located in `/etc/docker/`. If the file does not exist, create it first.  
The following example sets the log driver to `json-file` and sets the `max-size` and `max-file` options to enable automatic log-rotation.

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```
