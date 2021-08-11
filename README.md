# owntracks
Getting Owntracks Recorder Up and Running on Ubuntu 20.04.
Using snap for mosquitto, docker for the Recorder. nginx is native.
Why so complicated (and not baked all into one container)? Well, thats just what was available.

# Prerequisites
- Server running Ubuntu 20.04
- Having installed snap (`apt install snap`)
- Having installed docker (`apt install docker.io`) 
- Having installed ufw
- Having installed nginx (no container / snap)

# Setting up MQTT Broker (mosquitto) from snap Package
- There is no tls for mosquitto in this config. It'll be exclusively available to localhost. 
- `snap install mosquitto`
- Set username/password authentication: 
- config files (including password file) are located in `/var/snap/mosquitto/common`
- create password file, hash passwords `mosquitto_passwd -U mosquitto_pwd`
- `per_listener_settings true`
- `allow_anonymous false`
- `password_file /var/snap/mosquitto/common/mosquitto_pwd`
- restart it after configuration was changed: `snap restart mosquitto`
- Allow internal connections to MQTT broker (ufw): `ufw allow from 127.0.0.1 to 127.0.0.1 port 1183`
- Get logging info: `journalctl -f --unit snap.mosquitto*`
-

# Setting up Owntracks Recorder in Container (Docker)
- https://github.com/owntracks/recorder#docker
- `docker pull owntracks/recorder`
- `docker volume create recorder_store` (for permanent data storage)
- 
- Make it connect to MQTT: https://stackoverflow.com/questions/24319662/from-inside-of-a-docker-container-how-do-i-connect-to-the-localhost-of-the-mach (fixing "address not available")
- run it (correct networking mode, named "recorder"): `docker run -d --name recorder -p 8083:8083 -e OTR_HOST=127.0.0.1 -e OTR_PORT=1883 -e OTR_USER="<mqtt user name>" -e OTR_PASS="<mqtt password>" --network="host" -v recorder_store:/store owntracks/recorder`
- is it running? `docker ps`

# Setting up nginx Reverse Proxy for Access Protection to Owntracks
- `apt install apache2-utils` (for htpasswd files)
- create new user (DELETING previously existing info (`-c`)): `htpasswd -c /etc/nginx/.htpasswd <username>`  
- disable default site (if still up): `unlink /etc/nginx/sites-enabled/default`
- create config file `reverse-proxy.conf` in `/etc/nginx/sites-available`
- check for errors: `nginx -t`
- `service nginx reload`
- `ln -s /etc/nginx/sites-available/reverse-proxy.conf /etc/nginx/sites-enabled/reverse-proxy.conf`

# other stuff to watch out for:
- leave port 80 open for certbot/letsencrypt to be able to automatically renew certificates
