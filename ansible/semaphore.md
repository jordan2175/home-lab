# Setup Semaphore



## Create a semaphore system user

```
(
sudo bash
groupadd semaphore -r
useradd semaphore -r -m -s /bin/bash -g semaphore -G semaphore -c "Semaphore User"
usermod semaphore -L
)
```



## Install and secure the Database

```
(
apt-get update
apt-get install mariadb-server -y
mysql_secure_installation
)
```



## Configure Database

```
(
mariadb

CREATE DATABASE semaphore_db;
GRANT ALL PRIVILEGES ON semaphore_db.* TO semaphore@localhost IDENTIFIED BY 'randomly_generated_password_here';
exit
)
```

## Install Semaphore

Find the latest version at this URL: https://github.com/semaphoreui/semaphore/releases/latest

```
(
mkdir -p /local/src 
cd /local/src
wget https://github.com/semaphoreui/semaphore/releases/download/v2.12.14/semaphore_2.12.14_linux_arm64.deb
dpkg -i semaphore*.deb
)
```





