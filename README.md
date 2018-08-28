# Setting Up a Secure VPS

> **Pro tip:** Everywhere you see curly braces in these instructions, for example `{enter ip address here}`, the braces are placeholders indicating that there should a value there. Do not type in the braces.

## Prerequisites

1. Register a domain name with [Google Domains](https://domains.google.com/about/), [Gandi.net](http://www.gandi.net/) or other registrar. Look at `.com`, and `.me` domains for your personal site.
2. [Sign up](https://m.do.co/c/47e5e578d1cd) for a DigitalOcean account. You'll get a $10 credit, which let's you try it out for two months, for free.

## Getting ready

1. Follow the direction from Github's documentation to get yourself a shiny, [new SSH key](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/).
1. Add your SSH key to your DigitalOcean account. It's under `Settings > Security`. You get to settings from the gear icon in the upper-right corner of the navigation.
1. While you're there, set up 2FA for your DigitalOcean account. Yeah, I know 2FA seems like overkill... until someone uses a brute force attack to log into your account, change all of your security protocols, installs tons of malware on your domain, and then tries to blackmail you into getting it all back.
1. Create a Droplet. Select the $5/mo VPS type, accept all the defaults options they provide and **make sure you add your SSH key** by clicking that checkbox near the bottom of the screen. Then click the button to create your VM at the bottom.

## Accessing your VPS

Make sure that your private SSH key is currently loaded into your bash session by typing the following commands into your terminal.

```sh
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa  # If your SSH key name is different, use that
```

In your CLI, execute the command `ssh root@your.droplet.IP.address`. This will open up a secure shell connection to your droplet.

## Creating a user account

Your first step is to create an account for yourself.

1. Decide on a username
1. `mkdir /home/{ username }`
1. `mkdir /home/{ username }/.ssh`
1. `touch /home/{ username }/.ssh/authorized_keys`
1. `useradd { username } --home /home/{ username }`
1. `passwd { username }` then press enter.
1. You will be prompted to enter in the password for this account twice
1. Change the default shell with `chsh -s /bin/bash { username }`

### Account security

#### sudo

Allow new account to gain administrative privileges.

```
sudo visudo
```

This will open up a file in the Nano file editor, not vim. You can just start editing the file without the need to hit the `i` key.

Find the section where user privileges are specified. You should see a configuration section like this one. Add your new user account to be able to use `sudo`.

```
# User privilege specification
root    ALL=(ALL:ALL) ALL
{ username }   ALL=(ALL) ALL
```

After you've added that line in...

1. Hit `ctrl+x` to exit.
1. You'll be prompted to save the file, so just hit enter.
1. Then press `y` to verify the save.

### Transferring ownership

Set the new user as owner of the home directory: `chown -R { username } /home/{ username }`

#### Adding SSH key

1. Open up a new terminal instance so that you have a command line on your local computer.
1. In your shell, type `cat ~/.ssh/id_rsa.pub`. If you created a different SSH key name, type yours instead of `id_rsa.pub`.
1. Copy the entire public key that got printed to the shell.
1. Switch to your remote shell.
1. `vim /home/{username}/.ssh/authorized_keys`
1. Press `i`.
1. Paste in your public key.
1. Press `esc`
1. `:x` to save and quit.

## Using your account

1. Terminate your remote SSH session as root with the `exit` command.
1. Now create a new SSH session with your new account: `ssh { username }@{ ip address }`.

You are now logged into your DigitalOcean virtual machine on your user account. You're ready to start installing things.

## Base installs

1. Run the command `sudo apt-get update`
1. Install base packages `sudo apt-get install curl wget unzip git ufw nodejs npm nginx`
1. Install useful NPM packages: `sudo npm install -g grunt-cli http-server`

## Firewall

Now you'll set up ufw (uncomplicated firewall). Enter in the following commands.

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw allow www
sudo ufw enable
sudo ufw status verbose
```

Your server is now protected by a firewall that will deny **any** traffic other than SSH connections and web traffic.

# Getting Your Domain Pointed to Your Droplet

## Registrar DNS Steps

Once you have purchased your domain you can set it up to point at the server hosting your website. To do this you will need to tell your registrar where the server is to be resolved. There are three *name servers* that you will need to tell your registrar about. The GUI for each registrar is different, but you will need to enter the following on your registrar's control panel under name servers each on a new line.

`ns1.digitalocean.com`
`ns2.digitalocean.com`
`ns3.digitalocean.com`

## Pointing Web Traffic to Your Droplet

You will need to create custom records for your droplet on Digital Ocean, so go back to your Digital Ocean account and go to your *Networking* tab on the control panel and create a record there as well. From the tab go to the *Domains* tab and add a domain. To do this you will need to put your domain (i.e. `you.com`) in the domain field and pick a droplet to host it on.

You'll notice your new domain record will display beneath the creation form. Click on the domain entry and you'll see the DNS record form.

You'll need to add an **A** record for your domain, which is chosen by default on this form. In the *Enter name* field, enter in the value of **www** and in the *Enter IP Address* field, enter in your Droplet's IP address.

# Install & Configure Nginx

Nginx is a powerful web server that will allow you to serve your personal website, and your front-end capstone, from your new VPS. You already have nginx installed if you've followed the steps in this walk-through.

Digital Ocean has a [wonderful tutorial](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-16-04) showing you how to set it up.

> When you get to step 5 they show the directory where you clone your repo if it is a static personal site. `/var/www/html`

If you want to serve your Django, Rails, or Node server-side capstone from your VPS, then read below about seting up gunicorn with nginx.

## SSL and secure Nginx

There's a [Digital Ocean tutorial](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-14-04) that walks you through getting an SSL ceritificate from the Let's Encrypt* Certificate Authority. This is required for your web site to be trusted by web browsers, and use the `https` protocol for connections.

Then it shows you how to set up the Nginx web server to use that certificate.

# gunicorn and nginx setup for Django app

For example, if I wanted to deploy a Django REST Framework API project, I would create the following files.

## Setup

1. Install gunicorn with `sudo apt-get install gunicorn`
1. Clone your Django project into a sub-directory of your choosing in your home directory.
1. Copy the path to the project directory using `pwd`. You will need the path below.

## systemd gunicorn service

Create a service file for gunicorn that will keep the service running permanently.

> /lib/systemd/system/gunicorn.service

This is an example configuration that you would place in that file. The `WorkingDirectory` value is the directory that contains the entire Django project.

```
[Unit]
Description=gunicorn daemon
After=network.target

[Service]
User=chortlehoort
Group=www-data
WorkingDirectory=/full/path/to/django/project/directory
ExecStart=/full/path/to/django/project/directory/bin/gunicorn -w 3 --bind 127.0.0.1:8000 thenameoftheproject.wsgi
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

## nginx config for API

Next, you configure nginx to route requests coming in on a sub-domain to the Django application. Create the following file.

> /etc/nginx/sites-available/api

It would contain the following configuration. You would replace the `server_name` value with the sub-domain that you created with an `A` record for your domain.

```
server {
    listen 80;
    server_name api.replacewithyourdomain.com;
    access_log /var/log/nginx/api.log;

    location / {
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-NginX-Proxy true;

        include proxy_params;
        proxy_pass http://127.0.0.1:8000;
    }
}
```

> **Pro tip:** Make sure you make a symlink of the file in **sites-available** to the corresponding file in **sites-enabled**.
>
> `ln -s /etc/nginx/sites-available/api /etc/nginx/sites-enabled/api`