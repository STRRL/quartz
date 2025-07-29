---
id: wul0bj0xt6ag83dlnrnbung
title: Setup Fail2ban
desc: ""
updated: 1690036009215
created: 1690035913605
---

## Ubuntu 22.04

```bash
sudo apt update && sudo apt install -y fail2ban
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo sed -i "s/^\[sshd\]/[sshd]\nenabled=true/" /etc/fail2ban/jail.local
sudo systemctl enable --now fail2ban
```
