# devbox-setup
A collection of notes and scripts to make configuring new dev boxes faster and more automated.

## Ubuntu on Windows Subsystem for Linux:

### Set up X graphics
* Download and install [Xming](https://sourceforge.net/projects/xming/files/latest/download).
* Install libgtk and configure Xming as your X server. 
    ```
    sudo apt -y install libgtk2.0-0
    echo 'export DISPLAY=:0' >> ~/.bashrc
    ```
* When wrangling missing graphical dependencies, try `sudo apt -y install firefox`.
    * Firefox does a great job identifying and installing its depenencies, and it's a fairly elaborate graphical application.

## Ubuntu:

### Install frequently used tools
#### It is assumed that these commands are run in a Bash shell

* The script below installs:
    * `clang`
    * `curl`
    * `g++`
    * `git`
    * `gpg`
    * `grep`
    * `nano`
    * `nodejs`
    * `npm`
    * `vim`
    * `pip3`
    * `postgresql`
    * `rvm`
    * `ruby` (single-user install)
        * `bundler` (a ruby gem)
    * `screen`
    * `sed`
    * `sublime-text` (run using `subl`)
```
#####
# Install common dev tools from apt.
#####

# Add Sublime Text (stable) to apt
wget -qO - https://download.sublimetext.com/sublimehq-pub.gpg | sudo apt-key add -
sudo apt -y install apt-transport-https
echo "deb https://download.sublimetext.com/ apt/stable/" | sudo tee /etc/apt/sources.list.d/sublime-text.list
# Add nodejs (v6 LTS) to apt
curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
# Update apt
sudo apt update
# Install common dev tools
sudo apt -y install clang curl g++ git gnupg grep nano nodejs vim postgresql python3-pip screen sed sublime-text

#####
#Install RVM, Ruby, and Bundler for the current user.
#####

# Install RVM, ruby, and rubygems
gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
\curl -sSL https://get.rvm.io | bash -s stable --ruby
source /home/$USER/.rvm/scripts/rvm

# Install bundler
gem install bundler
```

* The script below adds these configurations:
    * Creates a directory for projects: `~/Projects`.
    * Adds new aliases to `~/.bashrc`:
        * `proj` changes the current directory to `~/Projects`.
        * `serv` starts a simple `python3` HTTP server in the current directory on port 4000.
        * `upgrade` performs an `apt` `update`, `upgrade`, and `autoremove`.
        * `python` and `pip` are pointed to `python3` and `pip3`.
```
####
#Create a Projects directory.
####

mkdir -p ~/Projects

####
#Add convenience shortcuts.
####

# A utility function to add aliased commands to one's .bashrc file.
# Takes two arguments, $ALIAS_NAME and $COMMAND_TEXT.
function add_alias_if_missing {
    ALIAS_NAME=$1;
    COMMAND_TEXT=$2;

    if [ -z "$ALIAS_NAME" ] || [ -z "$COMMAND_TEXT" ];
    then
        echo "add_alias_if_missing was provided invalid arguments: $ALIAS_NAME and $COMMAND_TEXT."
        echo "#####"
        exit;
    fi

    if ! grep -q "alias $ALIAS_NAME" ~/.bashrc;
    then
        echo "# This alias ('$ALIAS_NAME') was added by cjjeakle's ubuntu-tools setup script" >> ~/.bashrc
        echo "alias $ALIAS_NAME=\"$COMMAND_TEXT\"" >> ~/.bashrc
    fi
}

# Add any missing shortcuts.
add_alias_if_missing "upgrade" "sudo apt -y update && sudo apt -y upgrade && sudo apt -y autoremove"
add_alias_if_missing "proj" "cd ~/Projects/"
add_alias_if_missing "serv" "python3 -m http.server 4000"
add_alias_if_missing "python" "python3"
add_alias_if_missing "pip" "pip3"

# Load the newly added shortcuts.
source ~/.bashrc
```


### Git and GitHub setup
`sudo bash -c "$(wget -O - https://raw.githubusercontent.com/cjjeakle/devbox-setup/master/ubuntu-github --no-cache)"`

An example run:

```
touch password.txt && vim password.txt

cat password.txt | \
sudo bash -c "$(wget -O - https://raw.githubusercontent.com/cjjeakle/devbox-setup/master/ubuntu-github --no-cache)" \
-- -e "email@example.com" -n "Your Name" && \
rm password.txt
```

* Script arguments:
    * All arguments (except `-z`) are mandatory.
    * `stdin` The password you would like to protect the generated GitHub SSH key with. This password is provided via standard input.
    * `e <str>` (_E_mail) The email address you would like to use on your git commits, and would like to annotate your GitHub SSH key with.
    * `n <str>` (_N_ame) The full name you would like to use on your git commits.
    * `z` (_Z_oom) Deletes ~/.ssh/id_rsa and ~/.ssh/id_rsa.pub, if they exist, so ssh-keygen will not interrupt for write confirmation.
* Software installed:
    * `git`
* Configurations:
    * Sets global user.name and user.email for commits.
    * Generates an SSH key and attempts to save it to `~/.ssh/id_rsa`.
        * If `~/.ssh/id_rsa` already exists, `ssh-keygen` will interrupt the script and request write confirmation.
* Useful notes:
    * You can retrieve your generated ssh public key by running `cat ~/.ssh/id_rsa.pub` (it is also printed when the script completes).

### SSH setup
`sudo bash -c "$(wget -O - https://raw.githubusercontent.com/cjjeakle/devbox-setup/master/ubuntu-ssh --no-cache)"`

An example run:

```
sudo bash -c "$(wget -O - https://raw.githubusercontent.com/cjjeakle/devbox-setup/master/ubuntu-ssh --no-cache)" \
-- -u "login_user" -k "Your SSH Public Key Here"
```

* Script arguments:
    * All arguments are mandatory.
    * `k <str>` (_K_ey) the SSH public key you would like to log in against.
    * `u <str>` (_U_ser) the user you would like to use with SSH login.
* Software installed:
    * `openssh-server`
    * `ufw`
* Configurations:
    * Creates `/etc/ssh/sshd_config.factory-defaults` to back up `/etc/ssh/sshd_config`, if `/etc/ssh/sshd_config.factory-defaults` does not already exist.
    * Overwrites `~/.ssh/authorized_keys` to contain only the provided public key.
    * Disables SSH password authentication.
    * Sets the provided user as the only valid user for SSH login.
    * Enables rate limiting on login attempts.
* Useful notes:
    * An SSH public key will be requested.
        * [This page](https://www.digitalocean.com/community/tutorials/how-to-create-ssh-keys-with-putty-to-connect-to-a-vps) has some helpful reading on generating an OpenSSH key via PuTTygen (useful when using Windows).
