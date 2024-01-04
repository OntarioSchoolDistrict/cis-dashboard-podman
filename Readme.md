# Download files and setup podman
Setup working CIS area
```
mkdir /root/CIS
cd /root/CIS
```

Download CIS-CAT-Dashboard-v3.3.0-linux.zip into the CIS directory

Download CIS-SecureSuite-Product-License.zip into the CIS directory

Create a folder to work in:
```
mkdir /root/podman
cd /root/podman
vi Dockerfile
```
Insert or copy this code to the the Dockerfile
```
FROM ubuntu:22.04

RUN echo 'root:root' | chpasswd
RUN printf '#!/bin/sh\nexit 0' > /usr/sbin/policy-rc.d
RUN apt-get update
RUN apt-get install -y systemd systemd-sysv dbus dbus-user-session vim sudo unzip mariadb-server-10.6 openjdk-8-jdk-headless iproute2
RUN printf "systemctl start systemd-logind" >> /etc/profile

ENTRYPOINT ["/sbin/init"]
```
Build a temporary image
```
podman build -t temp-cis ./
```

Create some directories to instal CIS Dashboard into:
```
mkdir -p /srv/cis-cat/mariadb
mkdir -p /srv/cis-cat/usr_local_CCPD
```
# Install CIS Dashboard to temporary location
Run the temporary podman image created above
```
podman run -it --rm --name temp-cis -v /root/CIS:/root/CIS:Z  -v /srv/cis-cat/mariadb:/var/lib/mysql:Z -v /srv/cis-cat/usr_local_CCPD:/usr/local/CCPD:Z temp-cis
```

Log into the image using root:root
```
cd CIS
unzip CIS-CAT-Dashboard-v3.3.0-linux.zip
unzip CIS-SecureSuite-Product-Licens.zip
chmod u+x CCPD_unix_Installer.sh
```
Run the install using the instructions from CIS
```
./CCPD_unix_Installer.sh -c
```
The license file was extracted in the second command above.

When asked about /usr/local/CCPD already exists.

Answer Y to install to that directory anyway.

Exit the system after setup is complete
```
ctrl+P
ctrl+Q
```

# Cleanup temporary location
```
podman stop temp-cis
rm -fr /root/CIS
podman image rm temp-cis
```
# Setup a pod and the containers
Create the pod
```
podman pod create --name cis-cat --hostname cis-dashboard -p 80:80 -p 81:81 -p 443:443
```
Create the mariadb container, use the MYSQL password created for the Admin while running the installer from above.
```
podman run --pod cis-cat -d --restart unless-stopped --name mariadb -v /srv/cis-cat/mariadb:/var/lib/mysql -v /srv/cis-cat/usr_local_CCPD:/usr/local/CCPD -e MYSQL_ROOT_PASSWORD="<password from install>" mariadb:10.6
```
Create the CIS Dashboard container.
```
podman run --pod cis-cat -it --restart unless-stopped --name cis-cat-dashboard -v /srv/cis-cat/usr_local_CCPD:/usr/local/CCPD:Z -e CCPD_CONFIG_FILE=/usr/local/CCPD/conf/ccpd-config.yml -e MYSQL_ROOT_PASSWORD="<password from install>" ubuntu:22.04
```
Start the CIS Dashboard.
```
/usr/loca/CCPD/CIS-CAT_Pro_Dashboard start
```
Disconect from the container with ctrl+P and ctrl+Q

Start nginx-proxy-manager.
```
podman run --pod cis-cat -d --restart unless-stopped --name nginx-proxy -v /srv/cis-cat/nginx/data:/data:Z jc21/nginx-proxy-manager
```
