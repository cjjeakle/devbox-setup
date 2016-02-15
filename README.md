# devbox-setup
A collection of notes and scripts to make configuring new dev boxes faster and more automated.

## Set up an Ubuntu box:

### Install frequently used tools
`sudo bash -c "$(wget -O - https://raw.githubusercontent.com/cjjeakle/devbox-setup/master/ubuntu-tools)"`
* Installs:
    * `curl`
    * `git`
    * `clang`
    * `g++`
    * `nodejs`
    * `postgresql`
    * `screen`
    * `rvm`
    * `ruby` (2.1.0)
        * `bundler` (a ruby gem)
* Configurations:
    * Creates a directory for projects: `~/projects`
    * Adds commands to `~/.bashrc`<br/>
    1. `proj` changes the current directory to `~/projects`
    2. `serv` starts a simple python HTTP server in the current directory under port 4000

### Git and GitHub setup
`sudo bash -c "$(wget -O - https://raw.githubusercontent.com/cjjeakle/devbox-setup/master/ubuntu-github)"`
* Installs:
    * `git`
* Configurations:
    * Sets global user.name and user.email for commits
    * Generates an ssh key and attempts to save it to `~/.ssh/id_rsa`
        * If `~/.ssh/id_rsa` already exists, `ssh-keygen` will interrupt the script and request write confirmation

### SSH setup
`sudo bash -c "$(wget -O - https://raw.githubusercontent.com/cjjeakle/devbox-setup/master/ubuntu-ssh)"`
* Installs:
    * `openssh-server`
* Configurations:
    * Creates `/etc/ssh/sshd_config.factory-defaults` to back up `/etc/ssh/sshd_config`, if `/etc/ssh/sshd_config.factory-defaults` does not already exist
    * Overwrites `~/.ssh/authorized_keys` to contain only the provided public key
    * Disables ssh password authentication
    * Sets the provided user as the only valid user for ssh login
    * Enables rate limiting on login attempts
* Useful notes:
    * An SSH public key will be requested.<br/>
    [This page](https://www.digitalocean.com/community/tutorials/how-to-create-ssh-keys-with-putty-to-connect-to-a-vps) has some helpful reading on generating an OpenSSH key via PuTTygen (useful when using Windows).
