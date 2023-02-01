Install Dependencies for Staging
<a name="ubuntuinstall"></a>
### Installation instructions

**Install Nginx**:
```sh
apt install nginx
```
**NGINX Config**
```conf
server {
  listen       443 ssl;
  server_name  apistaging.joincommunion.xyz;
  ssl_certificate      /etc/letsencrypt/live/apistaging.joincommunion.xyz/fullchain.pem;
  ssl_certificate_key  /etc/letsencrypt/live/apistaging.joincommunion.xyz/privkey.pem;

  ssl_session_timeout  5m;

  ssl_protocols  TLSv1 TLSv1.1 TLSv1.2;
  ssl_prefer_server_ciphers   on;

  location / {
      proxy_pass  http://localhost:3000;
      proxy_set_header   Connection "";
      proxy_http_version 1.1;
      proxy_set_header        Host            $host;
      proxy_set_header        X-Real-IP       $remote_addr;
      proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header        X-Forwarded-Proto https;
  }
}

server {
    listen 80;
    server_name apistaging.joincommunion.xyz;
    return 301 https://$host$request_uri;
}
```

**Node.js & PM2 library globall**:

```sh
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash - &&\
sudo apt-get install -y nodejs
npm i -g pm2
```
**Config pm2 in project folder - put this config to ecosystem.config.js file in project path**
```js
module.exports = {
  apps: [
    {
      name: 'communion-staging',
      script: 'npm',
      args: 'run start', //'start'
    },
  ],
};
```

**Install Redis Server**:

```sh
apt install redis-server
```

**Install PostgreSQL**:

```sh
apt install postgresql
```

**Create DB user and DB, grant access**:
```txt
sudo -u postgres psql
postgres=# create database mydb;
postgres=# create user myuser with encrypted password 'mypass';
postgres=# grant all privileges on database mydb to myuser;
```
**Install github-actions self-hosted runner**:
```sh
# you can got this options, in project settings => actions => runners => add self-hosted runner
cd /opt
# Create a folder
mkdir actions-runner && cd actions-runner# Download the latest runner package
curl -o actions-runner-linux-x64-2.301.1.tar.gz -L https://github.com/actions/runner/releases/download/v2.301.1/actions-runner-linux-x64-2.301.1.tar.gz# Optional: Validate the hash
echo "3ee9c3b83de642f919912e0594ee2601835518827da785d034c1163f8efdf907  actions-runner-linux-x64-2.301.1.tar.gz" | shasum -a 256 -c# Extract the installer
tar xzf ./actions-runner-linux-x64-2.301.1.tar.gz
Configure
# Create the runner and start the configuration experience
./config.sh --url https://github.com/communion-dev/communion --token <token>
Install service
# run as simple user (non root)
sudo ./svc install
```

**setup cronjob for pm2**
```sh
#cd to project path and start PM2 service
@reboot cd /var/www/communion-staging/ && pm2 start
```
