mkdir /root/CIS
cd /root/CIS

Download CIS-CAT-Dashboard-v3.3.0-linux.zip into the CIS directory

Create a folder to work in:
mkdir /root/podman
cd /root/podman
vi Dockerfile
	FROM ubuntu:22.04

	RUN echo 'root:root' | chpasswd
	RUN printf '#!/bin/sh\nexit 0' > /usr/sbin/policy-rc.d
	RUN apt-get update
	RUN apt-get install -y systemd systemd-sysv dbus dbus-user-session vim sudo unzip mariadb-server-10.6 openjdk-8-jdk-headless
	RUN printf "systemctl start systemd-logind" >> /etc/profile

	ENTRYPOINT ["/sbin/init"]

podman build -t temp-cis ./

mkdir -p /srv/cis-cat/mariadb
mkdir -p /srv/cis-cat/usr_local_CCPD


podman run -it --rm -v /root/CIS:/root/CIS:Z  -v /srv/cis-cat/mariadb:/var/lib/mysql:Z -v /srv/cis-cat/usr_local_CCPD:/usr/local/CCPD:Z
