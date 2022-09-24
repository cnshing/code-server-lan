# code-server-lan
Setup VS Code Server securely over LAN through local-ip.co

## References
nginx reverse proxy - https://github.com/medic/nginx-local-ip<br/>
code-server setup - https://github.com/tteck/Proxmox

## Explanation
As code-server requires HTTPS for full functionality, users over LAN will normally have to configure a self-signed certificate on both clients and server. Fortunately, there exists already pre-made services to create SSL for devices in a local network hassle-free. This is done by using local-ip.co's DNS resolution to redirect traffic to our code-server with a public certificate, thus creating a secure connection between our client and code-server. our Since I run Proxmox to self-host my own code-server, I found [nginx-local-ip](https://github.com/medic/nginx-local-ip) and [tteck's](https://github.com/tteck/Proxmox) Proxmox helper scripts particularly helpful to quickly get code-server running as soon as possible.

## Requirements
Internet<br/>
docker-compose

## Install code-server
Run [tteck's](https://github.com/tteck/Proxmox)'s code-server script
```
bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/misc/code-server.sh)"
```
Note down instance IP and port at the end of installation as you will need it later.

## Setup HTTPS

```
git clone https://github.com/medic/nginx-local-ip.git
```

Modify the default nginx's configuration to enable WebSockets support at /nginx-local-ip/default.conf.templateby appending its location with
```
  location / {
  
    #Websocket
    proxy_set_header        Upgrade $http_upgrade;
    proxy_set_header        Connection upgrade;
    proxy_set_header        Accept-Encoding gzip;
    
    proxy_set_header        Host $host;
    proxy_set_header        X-Real-IP $remote_addr;
    proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header        X-Forwarded-Proto $scheme;

    proxy_pass              ${APP_URL}; # Running on Docker, localhost:port is not accessible
    proxy_read_timeout      900;        # Default 60 secs, 900 = 15 min => debugging purpose
  }
```

Finally build the docker container<br/>
By default, $(hostname -I | awk '{print $1}') should be the instance IP. Replace it with the instance IP noted earlier if that is not the case
```
APP_URL=http://$(hostname -I | awk '{print $1}'):8680 docker-compose up -d
```

## Connecting to your code-server
Connect to your code-server via https://ip-seperated-by-hypens.my.local-ip.co/<br/>
Example: https://192-168-0-3.my.local-ip.co

## Notes
Since our reverse proxy is installed in a docker container, this container must be running for us to securely connect to code-server.<br/>
It shold also be possible to set this network configuration without docker. 
