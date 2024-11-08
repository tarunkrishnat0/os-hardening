# Intro
## Target System(System to be hardened)
### Get Username and IP Address

```sh
# Get target-system-username
echo $USER

# Get target-system-ip
ip a
```
### Install dependencies
```sh
sudo apt update
sudo apt dist-upgrade

sudo apt install -y vim git tmux htop iputils-ping rsyslog fontconfig unzip curl nano
sudo apt install -y python3-dev python3-venv python3-virtualenv python3-pip libffi-dev gcc libssl-dev git net-tools openssh-server jq python3-pip sqlite-utils

sudo service ssh start
```

### Temporary Fixes: Apparmor
Apparmor has a bug that is resulting in hardening failure. Apply the patch as per this [PR](https://gitlab.com/apparmor/apparmor/-/merge_requests/1218/diffs)
```sh
sudo apt install apparmor apparmor-utils

# https://gitlab.com/apparmor/apparmor/-/merge_requests/1218/diffs
sudo gnome-text-editor /usr/lib/python3/dist-packages/apparmor/tools.py
```

## Local System(System with os-hardening repo)
### Install dependencies
```sh
git clone  --recursive -b dev https://github.com/tarunkrishnat0/os-hardening.git

sudo curl -L https://github.com/goss-org/goss/releases/latest/download/goss-linux-amd64 -o /usr/local/bin/goss
sudo chmod +rx /usr/local/bin/goss

# Install tools
sudo apt update
sudo apt dist-upgrade

sudo apt install -y vim git tmux htop iputils-ping rsyslog fontconfig unzip curl nano
sudo apt install -y python3-dev python3-venv python3-virtualenv python3-pip libffi-dev gcc libssl-dev git net-tools openssh-server jq python3-pip sqlite-utils

sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible
```
### Set ssh to login with keys instead of prompting for passwords

Generate ssh keys if you dont have one already
```sh
machine_id=$(cat /etc/machine-id)
ssh-keygen -t ed25519 -a 200 -C "${USER}@${machine_id}" -f ~/.ssh/id_ed25519
```

Copy the ssh key to the target system
```sh
ssh-copy-id target-system-username@target-system-ip

# SSH into the target system shouldnt prompt for password
ssh target-system-username@target-system-ip
```

# Hardening

## Update inventory.yml
Update the inventory.yml file with relevant entries
```sh
$ cat inventory.yml
all:
  children:
    clients:
      hosts:
        sys11:
          ansible_user: target-system-username
          ansible_host: target-system-ip
          ansible_connection: ssh
        sys12:
          ansible_user: ubuntu
          ansible_host: 192.168.10.12
          ansible_connection: ssh
  vars:
    ansible_python_interpreter: /usr/bin/python3
    ansible_ssh_key: ADD_SSH_KEY_HERE

```

Checking if all the systems are pingable
```sh
# Ping all systems
ansible all -m ping
```

## Running hardening playbook

```sh
ansible-playbook -i inventory.yml playbook.yml --ask-become-pass -k
```

## [Optional] Running Audit in the target system
```sh
# SSH into the system that you want to audit
ssh user@ip

# Run the below commands in the system that we are auditing
sudo curl -L https://github.com/goss-org/goss/releases/latest/download/goss-linux-amd64 -o /usr/local/bin/goss
sudo chmod +rx /usr/local/bin/goss

# Clone the repo
git clone -b benchmark-v2.0.0 https://github.com/tarunkrishnat0/UBUNTU22-CIS-Audit.git

# Run the audit
cd UBUNTU22-CIS-Audit
sudo ./run_audit.sh -v vars/CIS.yml -w Workstation
```

# Installing ClamAV
```sh
sudo apt-get install clamav clamav-daemon -y
sudo systemctl stop clamav-freshclam
sudo freshclam
sudo systemctl start clamav-freshclam
```

# Setting FIDO key for FDE
Refer [fido2-luks.md](Post-Ubuntu-Install/fido2-luks.md)