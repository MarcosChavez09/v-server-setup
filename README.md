# V-Server setup

## Description.
A repository with a step by step guide on how to set up a V-Server (Ubuntu based server) that walks you through, SSH Keys creation, disabling password authentication, installing NGINX web server and a custom HTML page, generating SSH Keys from the V-server to connect to Github.

## Table of Contents

- [1. Prerequisites](#prerequisites)
- [2. Clone this repository](#clone-repo)
- [3. Generate SSH Keys](#generate-ssh-keys)
- [4. Copy SSH Key Id to your server and login](#copy_ssh_key_id)
- [5. V-Server setup](v-server-setup.md)
- [6. Install NGINX](nginx-install)
- [7. Git Configuration](git-conf)
- [8. Additional Resources](#additional-resources)

## Prerequisites

Before setting up the server, make sure you have access to a v-server and you have the correpondent `ip_address`, `user_name` and `password` to login into.

- A V-Ubuntu server Ubuntu 20.04, 22.04 or 22.04.5 LTS
- A user with `sudo` rights
- Github account to connect via ssh keys
- linux command line knowledge

## Clone this repository

Clone this repository to your local machine and follow the README.md instruction along. 

Note: If you already have SSH Keys added to Github you can clone this repository using the ssh option. If you don't have a SSH Key yet, then just use the https://url_to_repo option. Link provided below.

[link to repository](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)

## Generate SSH Keys

if you haven´t created a SSH Key, type the following command in your terminal and give a name to it, that way you can easly know what is used for:

```
ssh-keygen -t ed25519 -C "your@email.com" -f ~/.ssh/name_of_your_key25519
```

When prompted for a passphrase, you can set one (for extra security) or press Enter for none. I suggest you give a passphrase.

This creates: 
- Private key: `~/.ssh/name_of_your_key25519`
- Public key: `~/.ssh/name_of_your_key25519.pub`

## Copy SSH Key id to your server and login

When created, copy your SSH Key Id and add it to the server using this command:

```
ssh-copy-id -i ~/.ssh/name_of_your_key25519.pub your_server_user_name@ip_server_address
```
- Replace `your_server_username` with the username you were given for the server.
- Enter your `password` when prompted (already provided to you, if not, ask your admin for it).

Test loggin with your key:
```
  ssh -i ~/.ssh/name_of_your_key25519 your_server_username@ip_server_address
```
- You should be able to log in without entering your password.

Once logged in, check for the authorized `ssh keys` in `ls -al ~/.ssh/authorized_keys` you should see your `ssh key` in there.

# V-Server setup

Deactivate the usage of password when loggin to the server. You have to change a line in the `ect/ssh/sshd_config` file.

Use admin rights (sudo) to perform this action
```
sudo nano /etc/ssh/sshd_config
```
Find the line `#PasswordAuthentication yes`, remove the `#` and change the value to `no`: 

```
#PasswordAuthentication yes -> PasswordAuthentication no
```
- Save and exit (`Ctrl+O`, `Enter`, `Ctrl+X` in nano) and restart the `ssh service` with:
```
 sudo systemctl restart ssh.service
```
Verify the deactivation of password required. Logout from the server and login again but with: 

```
ssh -i ~/.ssh/name_of_your_key25519 your_user_name@ip_server_addres 
```

You should be able to login without a `password`.

Test the login with password by using the following command:

```
ssh -o PubkeyAuthentication=no your_user_name@ip_server_addres
```
I should not be possible to to login with `user password`.

## Install NGINX

Install the nginx server in your `v-server` with:
```
sudo apt update
sudo apt install nginx -y
```
After the installation, verify the nginx service status.

```
systemctl status nginx.service
```
Now, create the directory `/alternatives` in the `/var/www/alternatives` and inside this new folder, create the file `alternate-index.html`
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

Restart nginx service and navigate to `http://your_v_server_ip:8081`. You shoulb be able to see your custom site on the web browser.

#  Git Configuration

Install Git on your server to interact with repositories (clone, pull, push, etc.).

```
sudo apt update
sudo apt install git -y
```

After that, configure git `user` and `email` on the V-Server. Git uses your name and email to record who makes each commit. This should match your GitHub identity for consistency and proper attribution.
```
git config --global user.name "Your Name"
git config --global user.email "your@email.com"
```

Now, we need to generate a new SSH Key pair on the V-Server, This key will be used by the server to authenticate with GitHub, allowing you to clone, pull, and push repositories from the server.
Note: Give a meaningfull name to this new key.
```
ssh-keygen -t ed25519 -C "your@email.com" -f ~/.ssh/meaningfull_name25519
```
- When prompted for a passphrase, you can set one (for extra security) or press Enter for none.
This creates:
- Private key: ~/.ssh/meaningfull_name25519
- Public key: ~/.ssh/meaningfull_name25519.pub

Add the SSH Key to the agent.
```
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/meaningfull_name25519
```
Then, add the pub key to GitHub.

```
  cat ~/.ssh/vserver25519.pub
```
- Copy the output.
- Go to GitHub SSH keys settings.
- Click New SSH key.
- Title it something like “V-Server meaningfull_name25519”.
- Paste the key and save.

Test the connection.
```
  ssh -T git@github.com
```
try cloning one of your repos:
```
  git clone git@github.com:your-username/your-repo.git
```