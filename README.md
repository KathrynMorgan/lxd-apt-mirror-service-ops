# CCIO Apt-Mirror LXD Server
## WARNING: apt-mirror uses up to hundreds of gigabytes for each OS & Release you configure!
This is an LXD container building repo to easily run any public or private ppa mirrors locally. The build includes a simple "curl pipe" install script to enroll new clients to use this repo once built.

#### 1. Launch Container
````sh
lxc launch ubuntu:bionic apt-mirror
````

#### 2. Exec into container shell
````sh
lxc exec apt-mirror bash
````

#### 3. Prep system
````sh
apt update
apt upgrade
apt install apache2 apt-mirror
mv /etc/apt/mirror.list /etc/apt/mirror.list.bak
rm /var/www/html/*
````

#### 4. Clone Repo
````sh
git clone https://gitlab.com/kat.morgan/apt-mirror-server.git /var/www/html/
````

#### 5. Configure apache
###### Default user:passwd is ccio:ccio. Remove with `htpasswd -D ccio`
````sh
ln -f /var/www/html/server/000-default.conf /etc/apache2/sites-enabled/000-default.conf
ln -f /var/www/html/server/apache2.conf /etc/apache2/apache2.conf
ln -f /var/www/html/server/htpasswd /etc/apache2/htpasswd
htpasswd -c /etc/apache2/htpasswd ${CHOOSE_USER_NAME}
apache2ctl restart
````

#### 6. Configure apt-mirror
````sh
ln -f /var/www/html/server/mirror.list /etc/apt/mirror.list
ln -f /var/www/html/repo/tools/run-apt-mirror /usr/bin/run-apt-mirror
````

#### 7. Run apt-mirror 
````sh
run-apt-mirror
````

#### 8. Visit the file server in your browser
  1. run "ip a" to find the ip address of your container
  2. type the ip address of your container in a browser window on the same lan
  3. enter "ccio:ccio" when prompted for credentials OR use the credentials you created
  4. click on `README` for instructions to enroll clients to use the new repo
  5. running `run-apt-mirror` the first time will consume significant bandwidth and may take many hours, subsequent runs to update the repo should be much shorter

## Storage Configuration
As an example, I dedicated a hard drive formatted with zfs to apt-mirror. To do so I created an lxc storage volume using flat directory driver at the zfs volume mount point:
````sh
    1. lxc storage create 4tb dir source=/mnt/zfs1/lxd  # create the storage volume
    2. lxc list storage                                 # to make sure the volume was created
    3. lxc profile copy default apt-mirror              # create a new lxd profile for apt-mirror
    4. lxc profile edit apt-mirror                      # change "default" to your new storage volume name
    -.                                                  # ie: I changed "default" to "4tb"
    5. lxc launch ubuntu:bionic apt-mirror -p apt-miror # create the container with the profile "apt-mirror"
````
