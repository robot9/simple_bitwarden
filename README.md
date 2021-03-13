# simple_bitwarden

minimal set up to run a small [bitwarden_rs](https://github.com/dani-garcia/bitwarden_rs) server in a small gcloud vm instance; 

It hides the bitwarden service under a sub-directory to avoid being exposed; 

The interface is only exposed via https protocol

## What's included:
### 1. bitwarden
The bitwarden_rs server
### 2. ddns
Periodically update domain name server
### 3.nginx
The reverse proxy to route traffic to the bitwarden service

## Instructions

1. install [docker-compose](https://docs.docker.com/compose/install/)
2. create your own config `cp .env.template .env`
2. apply for a free secondary domain if needed
   1. go to https://freemyip.com/ and pick a name, e.g. `domain`: `test.freemyip.com`
   2. write down the `password` and `domain` name 
3. create a `ddclient.conf` under `ddns` folder, example config:
```
daemon=1000                             # check every 300 seconds
syslog=yes                              # log update msgs to syslog
pid=/var/run/ddclient/ddclient.pid              # record PID in file.
ssl=yes                                 # use ssl-support.  Works with
use=web, web=checkip.dyndns.org/, web-skip='IP Address' # found after IP Address
protocol=freemyip,
# Your password
password=
# Replace with your domain
test.freemyip.com
```

4. Generate SSL certification for your `domain` to get `.key` and `.crt` file
    1. (better) follow https://letsencrypt.org/getting-started/ or simplely follow the commands here as it works on cloud VM
    
    ```
    # make sure 80 port is not occupied, 
    # if there are docker instance running, use `docker-compose down` to stop all of them, 
    # or `docker container stop <container_name>`
    sudo yum install certbot
    cd simple_bitwarden
    # Replace test.freemyip.com with your domain
    # Choose 1 to start local web service
    sudo certbot certonly -d test.freemyip.com
    ```
    you shold see files generated like 
    ```
    Congratulations! Your certificate and chain have been saved at:
    /etc/letsencrypt/live/test.freemyip.com/fullchain.pem
    Your key file has been saved at:
    /etc/letsencrypt/live/test.freemyip.com/privkey.pem
    ```
    now update the permission so that it can be accessed from nginx
    ```
    sudo chmod -R 755 /etc/letsencrypt/live/
    sudo chmod -R 755 /etc/letsencrypt/archive/
    ```
    
    2. create openssl self signed certs
5. Example `.env` config:
    
    ```
    # Your bitwarden instance `https://{domain}/{sub_dir}`
    DOMAIN=test.freemyip.com
    SUB_DIR=bitwarden
    SSL_KEY_PATH=/etc/letsencrypt/live/test.freemyip.com/privkey.pem
    SSL_CERT_PATH=/etc/letsencrypt/live/test.freemyip.com/fullchain.pem
    TZ=America/Los_Angeles
    
    # Please follow the instruction in .env.template to set up bitwarden
    SIGNUPS_ALLOWED=false
    ADMIN_TOKEN=
    
    PUID=1000
    PGID=994
    ```
6. run `docker-compose up -d`
7. visit your bitwarden instance `https://{domain}/{sub_dir}`, e.g. `https://test.freemyip.com/bitwarden/`

# Credits 
Copied from https://github.com/dadatuputi/bitwarden_gcloud