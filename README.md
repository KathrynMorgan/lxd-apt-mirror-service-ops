# CCIO Apt-Mirror LXD Server
## WARNING: apt-mirror uses up to hundreds of gigabytes for each OS & Release you configure!
LXD Container image configured for apt-mirror ppa repository mirroring

#### 0. Add BCIO Alpha LXD Images Repository (If not already in your remotes list)
````sh
lxc remote add bcio https://images.braincraft.io --public --accept-certificate
````

#### 2. Launch Container
````sh
lxc launch bcio:apt-mirror apt-mirror
````

#### 3. Launch apt-mirror first sync (This may take hours to complete)
````sh
lxc exec apt-mirror -- /bin/bash -c 'run-apt-mirror'
````

#### 4. Check apt-mirror IP address
````sh
lxc list
````

#### 5. Visit the apt-mirror IP in a browser on the same lan to view contents. Default credentials (admin:admin)
````sh
git clone https://gitlab.com/kat.morgan/apt-mirror-server.git /var/www/html/
````

#### 5. Configure webui credentials
````sh
lxc exec apt-mirror bash
htpasswd -D admin
htpasswd -c /etc/apache2/htpasswd ${CHOOSE_USER_NAME}
apache2ctl restart
````

## Storage Considerations:
#### A. Check Allocated Storage
````sh
# Check storage inventory
lxc storage list

# Confirm target storage pool has ~350GB or more available
lxc storage info ${STORAGE_VOLUME_NAME}

# If additional storage pool creation is required consult lxd help resources
# Or join #containercraft on IRC Freenode Network
````
#### B. Allocate a new storage pool
As an example, I dedicated a hard drive formatted with zfs to apt-mirror. To do so I created an lxc storage volume using flat directory driver at the zfs volume mount point:
````sh
    1. lxc storage create 4tb dir source=/mnt/zfs1/lxd  # create the storage volume
    2. lxc list storage                                 # to make sure the volume was created
    3. lxc profile copy default apt-mirror              # create a new lxd profile for apt-mirror
    4. lxc profile edit apt-mirror                      # change "default" to your new storage volume name
    -.                                                  # ie: I changed "default" to "4tb"
    5. lxc launch ubuntu:bionic apt-mirror -p apt-miror # create the container with the profile "apt-mirror"
````
