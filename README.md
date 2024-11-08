# Intro
```sh
git clone  --recursive -b benchmark-v2.0.0 https://github.com/tarunkrishnat0/os-hardening.git

sudo curl -L https://github.com/goss-org/goss/releases/latest/download/goss-linux-amd64 -o /usr/local/bin/goss
sudo chmod +rx /usr/local/bin/goss
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
