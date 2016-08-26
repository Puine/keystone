# Deployment to Ubuntu 14.04 with Nginx

Here is a quick guide to install everything you need on an Ubuntu server and deploy a KeystoneJS app on a domain name *www.my-domain.com* using **Nginx** and **pm2**.

## Prerequies
Before you begin this guide, you should have a regular, non-root user with sudo privileges configured on your server. You can learn how to configure a regular user account by following steps 1-4 of [this guide](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-14-04).

When you have an account available, log in as your non-root user to begin.

## Install everything you need
1. Install **make** and **git**
```
  sudo apt-get update
  sudo apt-get install -y gcc g++ make build-essential
  sudo apt-get install -y git
```
2. Check if you have an ssh key and add it on github
```
  cat ~/.ssh/id_rsa.pub
```
If the key doesn't exist, create one :
```
  ssh-keygen -t rsa -C "$your_email"
```
3. Install **nvm** ([source](https://github.com/creationix/nvm))
```
  curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.30.2/install.sh | bash
  source ~/.bashrc
```
4. Check the nodeJS last long-term support version [here](https://github.com/nodejs/LTS) and install it
```
  nvm install 4.x.x
```
5. Update **npm**
```
  npm install -g npm
```
6. Install **pm2** (to manage your nodeJS process)
```
  npm install pm2 -g
```
7. Install **nginx**
```
  sudo apt-get install nginx
```
8. Now, for KeystoneJS, we need **mongodb** :
```
  sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv EA312927
  echo "deb http://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list
  sudo apt-get update
  sudo apt-get install -y mongodb-org
```
9. ... and **yeoman**, and the yo keystone generator
```
  npm install -g yo
  npm install -g generator-keystone
```

## Start your project :
1. Now just create your project :
```
  mkdir my-test-project
  cd my-test-project
  yo keystone
```
2. Or clone it from your existing project in github and intall the npm dependencies :
```
  git clone MY-SUPER-PROJECT
  cd MY-SUPER-PROJECT
  npm install
```
3. Start the node process with **pm2**:
```
  pm2 start keystone.js
```
Now you have your nodeJS running. You need to link it to your domain name *www.my-domain.com*. We can use Nginx for that.

## Nginx Configuration

Nginx is the link between your domain name *www.my-domain.com* and your node process.

1. Create a nginx congif file named `www.my-domain.com` in `/etc/nginx/sites-available`.
The config file should look like this (change www.my-domain.com with your domain name):
```
  server {
          listen 80;
          listen [::]:80;
          server_name www.my-domain.com;
          location / {
                  proxy_pass http://localhost:3000;
          }
  }
```
You can change the port number (here : 3000) depending on where you run your nodeJS process.
2. Now create the following symbolic link :
```
  sudo ln -s /etc/nginx/sites-available/www.my-domain.com /etc/nginx/sites-enabled/.
```
3. and start nginx :
```
  sudo service nginx start
```
4. That's it ! You should see your beautiful keystone website on *www.my-domain.com*
