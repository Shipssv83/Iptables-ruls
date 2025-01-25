Hi, ![](https://user-images.githubusercontent.com/18350557/176309783-0785949b-9127-417c-8b55-ab5a4333674e.gif)my name is Serhii Shypylov
=========================================================================================================================================

-------------------------------

### Socials
<p align="left">
  <a href="https://t.me/oneitpro">
    <img src="https://img.icons8.com/ios-glyphs/30/ffffff/telegram-app.png" alt="Telegram" width="30" height="30" />
  </a>
  <a href="https://www.linkedin.com/in/sergey-shipilov-7262a31b4/">
    <img src="https://img.icons8.com/ios-glyphs/30/ffffff/linkedin.png" alt="LinkedIn" width="30" height="30" />
  </a>
  <a href="https://www.instagram.com/shipssvpl/">
    <img src="https://img.icons8.com/ios-glyphs/30/ffffff/instagram-new.png" alt="Instagram" width="30" height="30" />
  </a>
  <a href="https://www.facebook.com/profile.php?id=100083345006373">
    <img src="https://img.icons8.com/ios-glyphs/30/ffffff/facebook.png" alt="Facebook" width="30" height="30" />
  </a>
  <a href="https://discord.com/invite/6z5EyagDyW?ref=1it.pro">
    <img src="https://img.icons8.com/ios-glyphs/30/ffffff/discord.png" alt="Discord" width="30" height="30" />
  </a>
  <a href="mailto:admin@1it.pro">
    <img src="https://img.icons8.com/ios-glyphs/30/ffffff/new-post.png" alt="Mail" width="30" height="30" />
  </a>
  <a href="https://1it.pro/">
    <img src="https://img.icons8.com/ios-glyphs/30/ffffff/domain.png" alt="Website" width="30" height="30" />
  </a>
</p>
---

```bash
#!/bin/bash
echo "
 SSSSSSS  EEEEEEE  RRRRRR   HHH   HHH IIIIIIII IIIIIIII
 SSS      EEE      RRR  RR  HHH   HHH    III     III
 SSSSSS   EEEEE    RRRRRR   HHHHHHHHH    III     III
     SSS  EEE      RRR  RR  HHH   HHH    III     III
 SSSSSSS  EEEEEEE  RRR   RR HHH   HHH IIIIIIII IIIIIIII

 SSSSSSS  HHH   HHH YYY   YYY PPPPPP   YYY   YYY LLL      OOOOOOO  VVV     VVV
 SSS      HHH   HHH  YYY YYY  PPP  PP   YYY YYY  LLL      OOO OOO   VVV   VVV
 SSSSSS   HHHHHHHHH   YYYYY   PPPPPP     YYYYY   LLL      OOO OOO    VVV VVV
     SSS  HHH   HHH    YYY    PPP        YYY     LLL      OOO OOO     VVVV
 SSSSSSS  HHH   HHH    YYY    PPP        YYY     LLLLLLL  OOOOOOO      VV

      11   iii tttttttt      pppppp  rrrrrrr   ooooooo
      11         ttt         pp   pp rr   rr  oo     oo
      11   iii   ttt   ..... pppppp  rr  rrr  oo     oo
      11   iii   ttt         pp      rr   rr  oo     oo
      11   iii   ttt         pp      rr    rr  ooooooo
"
# Variables
MON_SERVER=Your_ip
JUMPBOX=Your_ip
VPN=Your_ip
IP_OFFICE=Your_ip

# Disable UFW if it is enabled
if sudo ufw status | grep -q "active"; then
    echo "Disabling UFW..."
    sudo ufw disable
fi

# Update and install iptables and iptables-persistent
sudo apt update
sudo apt install -y iptables iptables-persistent

# Flush all existing iptables rules
sudo iptables -F
sudo iptables -X
sudo iptables -t nat -F
sudo iptables -t nat -X
sudo iptables -t mangle -F
sudo iptables -t mangle -X

# Set default policies
sudo iptables -P FORWARD ACCEPT
sudo iptables -P OUTPUT ACCEPT

# Create WHITELIST chain
sudo iptables -N WHITELIST

# Allow local traffic
sudo iptables -I INPUT -i lo -j ACCEPT -m comment --comment 'Allowing local traffic'

# Allow traffic on ports 80 443 and 8443
sudo iptables -I INPUT 1 -p tcp --dport 80 -j ACCEPT
sudo iptables -I INPUT 1 -p tcp --dport 443 -j ACCEPT

# Allow established and related connections
sudo iptables -I INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT -m comment --comment 'Allowing established and related connections'

# Add specific rules to WHITELIST
sudo iptables -A WHITELIST -s $MON_SERVER -p tcp --dport 9100 -m comment --comment "Monitoring Grafana server" -j ACCEPT
sudo iptables -A WHITELIST -s $MON_SERVER -p tcp --dport 9080 -m comment --comment "Monitoring Promtail server" -j ACCEPT
sudo iptables -A WHITELIST -s $MON_SERVER -p tcp --dport 3100 -m comment --comment "Monitoring Loki server" -j ACCEPT
sudo iptables -A WHITELIST -s $MON_SERVER -p tcp --dport 10050 -m comment --comment "Monitoring Zabbix server" -j ACCEPT
sudo iptables -A WHITELIST -s $JUMPBOX -p tcp --dport 22 -m comment --comment "IP Jumpbox SSH" -j ACCEPT
sudo iptables -A WHITELIST -s $VPN -p tcp --dport 2053 -m comment --comment "IP VPN Pritunl in web 3x-ui" -j ACCEPT

# Apply WHITELIST chain to INPUT
sudo iptables -A INPUT -j WHITELIST

# Set default policies
sudo iptables -P INPUT DROP

sudo iptables -A INPUT -p tcp --syn -m limit --limit 10/s --limit-burst 20 -j ACCEPT
sudo iptables -A INPUT -p tcp --syn -j DROP
sudo iptables -A INPUT -p tcp --tcp-flags ALL NONE -j DROP
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -m conntrack --ctstate INVALID -j DROP

# Save and reload rules
sudo iptables-save > /etc/iptables/rules.v4
sudo netfilter-persistent save
sudo netfilter-persistent reload

# Display current iptables rules
#sudo iptables -L -v --line-numbers
sudo iptables -L INPUT --line-numbers
sudo iptables -L WHITELIST --line-numbers
sudo iptables -t nat -L --line-numbers
sudo iptables -S

echo "iptables rules configured."

```
