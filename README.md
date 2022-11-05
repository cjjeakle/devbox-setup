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
```
#####
# Install common dev tools from apt.
#####

# Update apt
sudo apt update
# Install common dev tools
sudo apt -y install avahi-daemon clang curl dirmngr g++ git gnupg gnupg2 grep nano nodejs npm openssl postgresql python3-pip screen sed vim

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

### Set some useful configurations
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

### Use screen to make shell sessions persistent
* Configure screen to use bash
    * Edit `~/.screenrc`:
    ```
    # ~/.screenrc
    defshell -bash
    caption always "%{= kc}Screen session %S on %H (system load: %l)%-28=%{= .m}%D %d.%m.%Y %0c"
    ```
* Add this to `~/.bashrc` to start a screen if there isn't one and attach to that screen when your session starts:
    ```
    DEFAULT_SCREEN_NAME=main
    if ! screen -list | grep -q ${DEFAULT_SCREEN_NAME}; then
        # If the default screen doesn't exist, create it
        screen -dmS ${DEFAULT_SCREEN_NAME}
    fi
    if [ -z $STY ]
    then
        # We're not in a screen, attach to the default screen
        screen -dr ${DEFAULT_SCREEN_NAME}
    fi
    ```

### Git and GitHub setup
```
# Make an SSH key to authenticate using
ssh-keygen -t ed25519 -C "your_email@example.com"

# Add this key to GitHub settings
cat .ssh/id_ed25519.pub

# Configure Git
git config --global user.name "FIRST_NAME LAST_NAME"
git config --global user.email "your_email@example.com"

# Verify configuration
git config --list
```

### SSH setup
```
# Log in as the user you'd like to ssh in as, populate the following vars
declare SSH_USER="";
declare SSH_PUBLIC_KEY="";

# Install openssh and back up the default config,
# based on the suggestions here: "https://help.ubuntu.com/community/SSH/OpenSSH/Configuring"
sudo apt -y install openssh-server ufw
sudo cp --no-clobber /etc/ssh/sshd_config /etc/ssh/sshd_config.factory-defaults
sudo chmod a-w /etc/ssh/sshd_config.factory-defaults

# Set the provided ssh public key as a login credential (Overwriting existing settings)
mkdir -p ~/.ssh
chmod 0700 ~/.ssh
echo $SSH_PUBLIC_KEY > ~/.ssh/authorized_keys
chmod 0644 ~/.ssh/authorized_keys

# Explicitly disable SSH password login using global regex substitution
sudo sed -i "s|#*PasswordAuthentication yes|PasswordAuthentication no|g" /etc/ssh/sshd_config
sudo sed -i "s|#*UsePAM yes|UsePAM no|g" /etc/ssh/sshd_config
if sudo grep -q "AllowUsers" /etc/ssh/sshd_config;
then
    sudo sed -i "s|AllowUsers.*|AllowUsers ${SSH_USER}|g" /etc/ssh/sshd_config
else
    echo "AllowUsers ${SSH_USER}" | sudo tee -a /etc/ssh/sshd_config
fi

# Rate limit connection attempts from a given IP address
sudo ufw limit ssh

# Restart the SSH service so changes take effect
sudo service ssh restart
```

* Useful notes:
    * An SSH public key is necessary (generated on the host you'd like to sign in from)
        * [This page](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent) has some helpful reading on generating an OpenSSH key.
