# V-Server setup

## Description
A repository with a step-by-step guide on how to set up a V-Server (Ubuntu-based server) that walks you through SSH key creation, disabling password authentication, installing the NGINX web server and a custom HTML page, and generating SSH keys from the V-Server to connect to GitHub.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Quick start](#quick-start)
3. [Generate SSH Keys](#generate-ssh-keys)
4. [Copy SSH Key ID to your server and login](#copy-ssh-key-id-to-your-server-and-login)
5. [Server setup](#server-setup)
6. [Install NGINX](#install-nginx)
7. [Git Configuration](#git-configuration)
8. [Project Checklist](#project-checklist)

## Prerequisites

Before setting up the server, make sure you have access to a V-Server and you have the corresponding `ip_address`, `user_name`, and `password` to log in.

- A user with `sudo` rights
- GitHub account to connect via SSH keys

## Quick start

Clone this repository to your local machine and follow the README.md instructions along.

Open your command line and type the following commands:

```
# With SSH configured (if SSH Keys provided to GitHub)

git clone git@github.com:MarcosChavez09/v-server-setup.git

# Classic https (if no SSH Keys provided to GitHub)

git clone https://github.com/MarcosChavez09/v-server-setup.git
```

## Generate SSH Keys

If you haven't created an SSH key, type the following command in your terminal and give a name to it, so you can easily know what it is used for:

```
ssh-keygen -t ed25519 -C "<your@email.com>" -f ~/.ssh/<name_of_your_key25519>
```

When prompted for a passphrase, you can set one (for extra security) or press Enter for none. I suggest you use a passphrase.

This creates:
- Private key: `~/.ssh/<name_of_your_key25519>`
- Public key: `~/.ssh/<name_of_your_key25519.pub>`

## Copy SSH Key ID to your server and login

When created, copy your SSH key ID and add it to the server using this command:

```
ssh-copy-id -i ~/.ssh/<name_of_your_key25519.pub> <your_server_user_name>@<ip_server_address>
```
- Replace `your_server_username` with the username you were given for the server.
- Enter your `password` when prompted (already provided to you, if not, ask your admin for it).

Test logging in with your key:
```
ssh -i ~/.ssh/<name_of_your_key25519> >your_server_username>@>ip_server_address>
```
- You should be able to log in without entering your password.

Once logged in, check for the authorized `ssh keys` with `ls -al ~/.ssh/authorized_keys`. You should see your `ssh key` in there.

## Server setup

Deactivate the usage of password when logging in to the server. You have to change a line in the `/etc/ssh/sshd_config` file.

Use admin rights (sudo) to perform this action:
```
sudo nano /etc/ssh/sshd_config
```
Find the line `#PasswordAuthentication yes`, remove the `#` and change the value to `no`:

```
-#PasswordAuthentication yes

+PasswordAuthentication no
```
- Save and exit (`Ctrl+O`, `Enter`, `Ctrl+X` in nano) and restart the `ssh service` with:
```
sudo systemctl restart ssh.service
```
Verify the deactivation of password requirement. Logout from the server and login again with:

```
ssh -i ~/.ssh/<name_of_your_key25519> <your_user_name>@<ip_server_address>
```

You should be able to login without a password.

Test the login with password by using the following command:

```
ssh -o PubkeyAuthentication=no <your_user_name>@<ip_server_address>
```
It should not be possible to login with the user password.

## Install NGINX

Install the NGINX server on your V-Server with:
```
sudo apt update
sudo apt install nginx -y
```
After the installation, verify the NGINX service status:

```
systemctl status nginx.service
```
Now, create the directory `/alternatives` in `/var/www/alternatives` and inside this new folder, create the file `alternate-index.html`:
```
sudo mkdir /var/www/alternatives
sudo nano /var/www/alternatives/alternate-index.html
```
- Add your custom HTML content to `alternate-index.html`
- Save and exit (`Ctrl+O`, `Enter`, `Ctrl+X` in nano).

Add configuration for the new `alternate-index` file by creating a new file in `/etc/nginx/sites-enabled/alternatives` and add the following config:
```
sudo nano /etc/nginx/sites-enabled/alternatives
```
The content:
```
server {
    listen 8081;
    listen [::]:8081;

    root /var/www/alternatives;
    index alternate-index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```
- Save and exit (`Ctrl+O`, `Enter`, `Ctrl+X` in nano).

Restart the NGINX service and navigate to `http://<your_v_server_ip>:8081`. You should be able to see your custom site in the web browser.

## Git Configuration

Install Git on your server to interact with repositories (clone, pull, push, etc.).

```
sudo apt update
sudo apt install git -y
```

After that, configure Git `user` and `email` on the V-Server.
```
git config --global user.name "<Your Name>"
git config --global user.email "<your@email.com>"
```

Now, we need to generate a new SSH key pair on the V-Server. This key will be used by the server to authenticate with GitHub, allowing you to clone, pull, and push repositories from the server.
Note: Give a meaningful name to this new key.
```
ssh-keygen -t ed25519 -C "<your@email.com>" -f ~/.ssh/<meaningful_name25519>
```
- When prompted for a passphrase, you can set one (for extra security) or press Enter for none.
This creates:
- Private key: `~/.ssh/<meaningful_name25519>`
- Public key: `~/.ssh/<meaningful_name25519.pub>`

Add the SSH key to the agent:
```
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/<meaningful_name25519>
```
Then, add the public key to GitHub:

```
cat ~/.ssh/<meaningful_name25519.pub>
```
- Copy the output.
- Go to GitHub SSH keys settings.
- Click New SSH key.
- Title it something like "V-Server <meaningful_name25519>".
- Paste the key and save.

Test the connection:
```
ssh -T git@github.com
```
Try cloning one of your repos:
```
git clone git@github.com:MarcosChavez09/v-server-setup.git
```
## Project Checklist

- 📄 [Checklist (PDF)](docs/checklist.pdf)