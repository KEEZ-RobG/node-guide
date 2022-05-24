# Remote SSH Access - Windows (PuTTy)

This is a guide to setup a Lukso validator node in home environment. The guide suggests a use of a dedicat

> **_NOTE:_** Most of the steps require working in a terminal

  - [Remote Access](#remote-access)
    - [Install SSH](#install-ssh)
    - [Configure SSH](#configure-ssh)
    - [Configure Firewall](#configure-firewall)
    - [Enable SSH](#enable-ssh)
    - [Resolve Hostname](#resolve-hostname)
    - [Disable Password Authentication](#disable-password-authentication)
    - [Disable Non-Key Remote Access](#disable-non-key-remote-access)
    - [Verify Remote Access](#verify-remote-access)
  - [Keep System Up to Date](#keep-system-up-to-date)
  - [Disable Root Access](#disable-root-access)
  - [Block Unathorised Access](#block-unathorised-access)
  - [Configure Firewall](#configure-firewall)
  - [Improve SSH Connection](#improve-ssh-connection)




### Remote Access

SSH allows access to the command line of your node from a personal computer. The first section of the guide provides instructions for enabling access within your local network. This is a common practice and useful if a node machine does not have inputs (keyboard/mouse) or a display. Once set up, a node machine can be placed elsewhere and controlled/maintained by a personal computer.

The second section provides instructions for enabling access to your node from a remote location outside your local network. These steps are optional. The guide endeavors to follow best security practices, but be aware that allowing outside access does weaken security.

### Node Configuration

#### Install SSH Server

The steps in this section are performed in the terminal of your node machine

```shell=
sudo apt install --assume-yes openssh-server
```

#### Confiugre SSH

Choose a port number between than `50000` and `65000`. This will be used later.

```shell=
sudo nano /etc/ssh/sshd_config
```
Find the line that says: `#Port 22`

Change and enable a port by uncommenting (removing `#`) and changing `22` to new chosen port number.

Example: `Port 50000`

Save and close editor by pressing `CTRL` + `X`, then type `Y`, and hit `ENTER`.

#### Configure Firewall

Enable ssh in firewall by replacing *replace-port* with the port number chosen in the last step:

```shell=
sudo ufw allow replace-port
```

#### Enable SSH

```shell=
sudo systemctl start ssh
sudo systemctl enable ssh
```

#### Resolve Hostname

You must know the local IP address of your node machine in you local network, it requires IP. Execute following command to resolve a node machine's IP:

```shell=
ifconfig
```

Locate IP  address (`inet`) in `eno1` section, e.g. `192.168.86.29`.

```
eno1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
      inet 192.168.86.29  netmask 255.255.255.0  broadcast 192.168.86.255
```

Close ssh session by executing `exit`.

> **_NOTE:_** Following steps are performed a personal computer.

Verify basic access to a node machine by using ssh. SSH requires user name of a node machine, its hostname and previously chosen ssh port.

```shell=
vim ~/.ssh/config 
```

Type in the following and replace *replace-user*, *replace-ip*, and *replace-port*:

```shell=
Host lukso
  User replace-user
  HostName replace-ip
  Port replace-port
```

Attempt to connect to verify the configuration:

```shell=
ssh lukso
```

Once connected, enter a password of user on a node machine. If a connection was okay, a shell should be presented in a terminal. At this point, it could closed.

#### Disable Password Authentication

On a personal computer, create new key pair for ssh authentication if needed.

```shell=
ssh-keygen -t rsa -b 4096
```

Copy a generated public key **keyname.pub** to a node machine. Replace **keyname.pub** with a key in home directory.

```shell=
ssh-copy-id -i ~/.ssh/keyname.pub lukso
```

#### Disable Non-Key Remote Access

On a personal computer, try to ssh again. This time it should not prompt for a password.

```shell=
ssh lukso
```

Configure SSH by opening a configuration file and modifying several options:

```shell=
sudo vim /etc/ssh/sshd_config
```

Options:

```shell=
ChallengeResponseAuthentication no
PasswordAuthentication no
PermitRootLogin prohibit-password
PermitEmptyPasswords no
```

Save and close editor by pressing `SHIFT` + `:`, then type `wq`, and hit enter. Validate SSH configuration and restart ssh service.

```shell=
sudo sshd -t
sudo systemctl restart sshd
```

Close ssh session by executing `exit`.

#### Verify Remote Access

```shell=
ssh lukso
```
    
Stay connected to a remote node machine to perform next steps.

### Keep System Up to Date
    
Update a system manually:
    
```shell=
sudo apt-get update -y
sudo apt dist-upgrade -y
sudo apt-get autoremove
sudo apt-get autoclean
```

Keep a system up to date automatically:

```shell=
sudo apt-get install unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades
```

### Disable Root Access
    
A root access should not be used. Instead, a user should be using `sudo` to perform privilged operations on a system.

```shell=
sudo passwd -l root
```

### Block Unathorised Access

Install `fail2ban` to block IP addresses that exceed failed ssh login attempts.

```shell=
sudo apt-get install fail2ban -y
```

Edit a config to monitor ssh logins

```shell=
sudo vim /etc/fail2ban/jail.local
```

Replace *replace-port* to match the ssh port number.

```shell=
[sshd]
enabled=true
port=replace-port
filter=sshd
logpath=/var/log/auth.log
maxretry=3
ignoreip=
```

Save and close editor by pressing `SHIFT` + `:`, then type `wq`, and hit enter. Restart `fail2ban` service:

```shell=
sudo systemctl restart fail2ban
```

### Configure Firewall

By default deny all traffic:

```shell=
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

Allow P2P ports for Lukso clients:

```shell=
sudo ufw allow 30303/tcp
sudo ufw allow 8545/tcp
sudo ufw allow 8598/tcp
sudo ufw allow 8080/tcp
sudo ufw allow 3500/tcp
sudo ufw allow 4000/tcp
sudo ufw allow 13000/tcp
sudo ufw allow 12000/udp
sudo ufw allow 30303/udp
```

> **_NOTE:_** make sure to open same ports on your home router

Enable Firewall:

```shell=
sudo ufw enable
```

Verify firewall configuration:

```shell=
sudo ufw status numbered
```

It should look something like this (may be missing some ports):

```shell=
Status: active

     To                         Action      From
     --                         ------      ----
[ 1] 13000/tcp                  ALLOW IN    Anywhere               
[ 2] ssh-port/tcp               ALLOW IN    Anywhere               
[ 3] 12000/udp                  ALLOW IN    Anywhere               
[ 4] 9090/tcp                   ALLOW IN    Anywhere              
[ 5] 3000/tcp                   ALLOW IN    Anywhere               
[ 6] 13000/tcp (v6)             ALLOW IN    Anywhere (v6)          
[ 7] ssh-port/tcp (v6)          ALLOW IN    Anywhere (v6)          
[ 8] 12000/udp (v6)             ALLOW IN    Anywhere (v6)          
[ 9] 9090/tcp (v6)              ALLOW IN    Anywhere (v6)          
[10] 3000/tcp (v6)              ALLOW IN    Anywhere (v6)
```

### Improve SSH Connection

While setting up a system, ssh terminal may seem to be slow due wifi power management settings on a node machine. To disable it, modify a config.

```shell=
sudo vim /etc/NetworkManager/conf.d/default-wifi-powersave-on.conf
```

Config:
```shell=
[connection]
wifi.powersave = 2
```

Save and close editor by pressing `SHIFT` + `:`, then type `wq`, and hit enter. Restart `NetworkManager` service:

```shell=
sudo systemctl restart NetworkManager
```

