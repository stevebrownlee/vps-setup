# Setting Up a Secure VPS

> **Pro tip:** Everywhere you see curly braces in these instructions, for example `{enter ip address here}`, the braces are placeholders indicating that there should a value there. Do not type in the braces.

## Prerequisites

1. Register a domain name with [Google Domains](https://domains.google.com/about/), [Gandi.net](http://www.gandi.net/) or other registrar. Look at `.com`, and `.me` domains for your personal site.
2. [Sign up](https://www.digitalocean.com/?refcode=47e5e578d1cd) for a DigitalOcean account.

## Getting ready

1. Follow the direction from Github's documentation to get yourself a shiny, [new SSH key](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/).
1. Add your SSH key to your DigitalOcean account. It's under `Settings > Security`. You get to settings from the gear icon in the upper-right corner of the navigation.
1. While you're there, set up 2FA for your DigitalOcean account. Yeah, I know 2FA seems like overkill... until someone uses a brute force attack to log into your account, change all of your security protocols, installs tons of malware on your domain, and then tries to blackmail you into getting it all back.
1. Create a Droplet. Select the $5/mo VPS type, accept all the defaults options they provide and **make sure you add your SSH key** by clicking that checkbox near the bottom of the screen. Then click the button to create your VM at the bottom.

## Accessing your VPS

Make sure that your private SSH key is currently loaded into your bash session by typing the following commands into your terminal.

```
eval `ssh-agent`
ssh-add ~/.ssh/id_rsa
```

In your CLI, execute the command `ssh root@{your droplet IP address}`. This will open up a secure shell connection to your droplet.

#### If you are prompted for the root password follow the directions in this section, otherwise, skip down to creating a user account

> **Attention:** If you were not prompted for your password, ignore this section. Seriously. Move along. I'm not kidding.

You will need to go back to the Digital Ocean site and click on your Droplet, and then the "Reset root password" button. They will email you a new root password.

After you type that in, you will gain access. Yay!

Now type `exit` to log out and then execute the following command to add your public key to your remote machine. This will use SSH to handle authentication and you'll never need to enter a password again for **root**.

```
cat ~/.ssh/id_rsa.pub | ssh root@{droplet ip address} "cat >> ~/.ssh/authorized_keys"
```

## Creating a user account

Your first step is to create an account for yourself.

1. Decide on a username
1. `mkdir /home/{ username }`
1. `mkdir /home/{ username }/.ssh`
1. `touch /home/{ username }/.ssh/authorized_keys`
1. `useradd { username } --home /home/{ username }`
1. `passwd { username }` and you'll need to enter in the password for the user twice
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

Open up a new terminal instance so that you have a command line on your local computer. Then execute the following command. This copies your public key from your local machine to the droplet.

```
cat ~/.ssh/id_rsa.pub | ssh {username you created above}@{droplet ip address} "cat >> ~/.ssh/authorized_keys"
```


## Using your account

1. Terminate your remote SSH session as root with the `exit` command.
1. Now create a new SSH session with your new account: `ssh { username }@{ ip address }`.

You are now logged into your DigitalOcean virtual machine on your user account. You're ready to start installing things.

## Base installs

1. Run the command `sudo apt-get update`
1. Install base packages `sudo apt-get install curl wget unzip git ufw nodejs npm nginx`
1. Create symlink for Node: `sudo ln -s /usr/bin/nodejs /usr/bin/node`
1. Install useful NPM packages: `sudo npm install -g nvm bower grunt-cli http-server express express-generator pm2`

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

Nginx is a powerful web server that will allow you to serve your applications from your new VPS. You already have nginx installed if you've followed the steps in this walk-through.

Digital Ocean has a [wonderful tutorial](https://www.digitalocean.com/community/tutorials/how-to-configure-the-nginx-web-server-on-a-virtual-private-server) showing you how to set it up.

## SSL and secure Nginx

There's a [Digital Ocean tutorial](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-14-04) that walks you through getting an SSL ceritificate from the Let's Encrypt* Certificate Authority. This is required for your web site to be trusted by web browsers, and use the `https` protocol for connections.

Then it shows you how to set up the Nginx web server to use that certificate.
