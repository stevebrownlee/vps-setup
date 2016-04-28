# Setting yourself up on a VPS

## Prerequisites

1. Register a domain name with [Google Domains](https://domains.google.com/about/), [Gandi.net](http://www.gandi.net/) or other registrar. Look at `.com`, and `.me` domains for your personal site.
2. [Sign up](https://www.digitalocean.com/?refcode=47e5e578d1cd) for a DigitalOcean account.

## Getting ready

1. Follow the direction from Github's documentation to get yourself a shiny, [new SSH key](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/).
1. Add your SSH key to your DigitalOcean account. It's under `Settings > Security`. You get to settings from the gear icon in the upper-right corner of the navigation.
1. While you're there, set up 2FA for your DigitalOcean account. Yeah, I know 2FA seems like overkill... until someone uses a brute force attack to log into your account, change all of your security protocols, installs tons of malware on your domain, and then tries to blackmail you into getting it all back.
1. Create a Droplet. Select the $5/mo VPS type, accept all the defaults options they provide and **make sure you add your SSH key** by clicking that checkbox near the bottom of the screen. Then click the button to create your VM at the bottom.

## Accessing your VPS

Make sure that your private SSH key is currently loaded into your bash session.

```
eval `ssh-agent`
ssh-add ~/.ssh/id_rsa
```

In your CLI, execute the command `ssh root@[your droplet IP address but not the brackets]`. This will open up a secure shell connection to your droplet.

#### If you are prompted for the root password

> **Note:** If you were not prompted for your password, ignore this section

You will need to go back to the Digital Ocean site and click on your Droplet, and then the "Reset root password" button. They will email you a new root password.

After you type that in, you will gain access. Yay!

Now type `exit` to log out and then execute the following command to add your public key to your remote machine. This will use SSH to handle authentication and you'll never need to enter a password again for **root**.

```
cat ~/.ssh/id_rsa.pub | ssh root@[your.ip.address.here] "cat >> ~/.ssh/authorized_keys"
```

## Creating a user account

Your first step is to create an account for yourself.

1. Decide on a username
1. `mkdir /home/{{ username }}`
1. `mkdir /home/{{ username }}/.ssh`
1. `touch /home/{{ username }}/.ssh/authorized_keys`
1. `useradd {{ username }} --home /home/{{ username }}`
1. `passwd {{ username}}` and you'll need to enter in the password for the user twice
1. Change the default shell with `chsh -s /bin/bash {{ username }}`

### Account security

#### sudo

Allow new account to gain administrative privileges.

```
sudo visudo
```

This will open up a file in the Nano file editor, not vim. You cna just start editing the file without the need to hit the `i` key.

Find the section where user privileges are specified. You should see a configuration section like this one. Add your new user account to be able to use `sudo`.

```
# User privilege specification
root    ALL=(ALL:ALL) ALL
{{ username }}   ALL=(ALL) ALL
```

After you've added that line in...

1. Hit `ctrl+x` to exit.
1. You'll be prompted to save the file, so just hit enter.
1. Then press `y` to verify the save.

### Transferring ownership

Set the new user as owner of the home directory: `chown -R {{ username }} /home/{{ username }}`

#### Adding SSH key

Open up a new terminal instance so that you have a command line on your local computer. Then execute the following command. This copies your public key from your local machine to the droplet.

```
cat ~/.ssh/id_rsa.pub | ssh [username you created above]@[your.ip.address.here] "cat >> ~/.ssh/authorized_keys"
```


## Using your account

1. Terminate your remote SSH session as root with the `exit` command.
1. Now create a new SSH session with your new account: `ssh {{ username }}@{{ ip }}`.

You are now logged into your DigitalOcean virtual machine on your user account. You're ready to start installing things.

## Base installs

1. Run the command `sudo apt-get update`
1. Install base packages `sudo apt-get install curl wget unzip git ufw redis-server nodejs npm`
1. Create symlink for Node: `sudo ln -s /usr/bin/nodejs /usr/bin/node`
1. Install useful NPM packages: `sudo npm install -g nave bower grunt-cli http-server express express-generator pm2`

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




