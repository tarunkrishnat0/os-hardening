# Intro
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

# Hardening

Update the inventory.yml file with relevant entries

Checking if all the systems are pingable
```sh
# Ping all systems
ansible all -m ping
```

# Running hardening playbook

```sh
ansible-playbook -i inventory.yml playbook.yml --ask-become-pass -k
```
