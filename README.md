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

In your CLI, execute the command `ssh root@[your droplet IP address but not the brackets]`. This will open up a secure shell connection to your droplet.

You will be prompted for the root password, which likely was never sent to you, so you will need to go back to the Digital Ocean site and click on your Droplet, and then the "Reset root password" button. They will email you a new root password.

After you type that in, you will gain access. Yay!

Now type `exit` to log out and then execute the following command to add your public key to your remote machine. This will use SSH to handle authentication and you'll never need to enter a password again for **root**.

```
cat ~/.ssh/id_rsa.pub | ssh root@[your.ip.address.here] "cat >> ~/.ssh/authorized_keys"
```

## Creating a user account

Your first step is to create an account for yourself.

1. Decide on a username
1. `mkdir /home/{{ username }}`
1. `useradd {{ username }} --home /home/{{ username }}`
1. `passwd {{ username}}` and you'll need to enter in the password for the user twice
1. Change the default shell with `chsh -s /bin/bash {{ username }}`

### Account security

#### Adding SSH key

On your host machine, execute the following command.

```
cat ~/.ssh/id_rsa.pub | ssh [username you created above]@[your.ip.address.here] "cat >> ~/.ssh/authorized_keys"
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

Save the file with `esc`, `:x`.

### Transferring ownership

Set the new user as owner of the home directory: `chown -R {{ username }} /home/{{ username }}`


## Using your account

Terminate your SSH session as root with the `exit` command. Then you can try to create a new SSH session with your new account: `ssh {{ username }}@{{ ip }}`.

## Base installs

1. Run the command `sudo apt-get update`
1. Install base packages `sudo apt-get install curl wget unzip git ufw redis-server nodejs npm`
1. Create symlink for Node: `sudo ln -s /usr/bin/nodejs /usr/bin/node`
1. Install useful NPM packages: `sudo npm install -g nave bower grunt-cli http-server express express-generator sails pm2`

## Firewall

Now you'll set up ufw (uncomplicated firewall). Follow [the instructions](https://www.digitalocean.com/community/tutorials/how-to-setup-a-firewall-with-ufw-on-an-ubuntu-and-debian-cloud-server) on the DigitalOcean Community page. We're going to open up ports 22 and 80.




