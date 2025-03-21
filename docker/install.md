# Installation

The method I used to install the docker engine on a Raspberry Pi was found in the **Install using the apt repository** section of this site: https://docs.docker.com/engine/install/ubuntu/

## Uninstall all existing packages

`for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done`


## Set up Docker's apt repository.

### Add Docker's official GPG key:

`
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
`

### Add the repository to Apt sources:

`
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
`

### Verify installation

`
sudo docker run hello-world
`
