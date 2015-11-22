# Setting yourself up on a VPS

## Prerequisites

1. Register a domain name with [Google Domains](https://domains.google.com/about/), [Gandi.net](http://www.gandi.net/) or other registrar. Look at `.com`, and `.me` domains for your personal site.
2. [Sign up](https://www.digitalocean.com/pricing/) for a DigitalOcean account, and purchase a $5/month VPS.

## Getting ready

1. Get yourself a shiny, [new SSH key](https://help.github.com/articles/generating-ssh-keys/). You can use an SSH key to connect to Github, as well as your VPS.
1. When setting up your Droplet, pick the *Ubuntu 14.04 x64* image to be the base operating system.
2. Add your SSH key to your DigitalOcean account. It's under `Settings > Security`.
2. While you're there, set up 2FA for your DigitalOcean account. Yeah, I know 2FA seems like overkill... until someone uses a brute force attack to log into your account, change all of your security protocols, installs tons of malware on your domain, and then tries to blackmail you into getting it all back.

## Accessing your VPS

Make sure that your private SSH key is currently loaded into your bash session.

```
eval `ssh-agent`
ssh-add ~/.ssh/id_rsa
```

In your CLI, execute the command `ssh root@{{ your droplet IP address }}`. This will open up a secure shell connection to your droplet. 

## Creating a user account

Your first step is to create an account for yourself.

1. Decide on a username
1. `mkdir /home/{{ username }}`
1. `useradd {{ username }} --home /home/{{ username }}`
1. `passwd {{ username}}` and you'll need to enter in the password for the user twice
1. Change the default shell with `chsh -s /bin/bash {{ username }}`

### Account security

#### Adding SSH key

Create another terminal window. Open up your public key file that got created when you created your SSH key on your local machine.

```
cat ~/.ssh/id_rsa.pub
```

Copy what got output to the console.

```
mkdir /home/{{ uername }}/.ssh
vi /home/{{ username }}/.ssh/authorized_keys
```

#### sudo

Allow new account to gain administrative privileges.

```
sudo visudo
```

Then find the section where user privileges are specified. You should see a configuration section like this one. Add your new user account to be able to use `sudo`.

```
# User privilege specification
root    ALL=(ALL:ALL) ALL
{{ username }}   ALL=(ALL) ALL
```

Save that file and then stop your SSH session by executing the command `exit`.

Then you can connect to your VPS with your

## Base installs

1. Install base packages `sudo apt-get curl wget git ufw redis-server nodejs npm`
1. Create symlink for Node: `ln -s /usr/bin/nodejs /usr/bin/node`
1. Install useful NPM packages: `sudo npm install -f nave bower grunt-cli http-server`


