#conteneur Apache
systemctl status apache2
ip a
ip route
ping -c 3 172.16.0.1
getent hosts deb.debian.org
curl -I http://deb.debian.org

cat > /var/www/html/index.html <<'EOF'
<!DOCTYPE html>
<html lang="fr"><head><meta charset="utf-8"><title>Vikor - Serveur Web DMZ</title></head>
<body><h1>Société Vikor</h1><p>Serveur Web Apache2 hébergé en DMZ (172.16.0.10)</p></body></html>
EOF
curl http://localhost

ping 192.168.20.38

#Agent Zabbix (Apache)

apt update
apt install -y zabbix-agent2
sed -i 's/^Server=.*/Server=192.168.10.20/; s/^ServerActive=.*/# ServerActive=/; s/^Hostname=.*/Hostname=ApacheWeb/' /etc/zabbix/zabbix_agent2.conf
systemctl restart zabbix-agent2
systemctl enable zabbix-agent2
ss -tlnp | grep 10050
grep -E '^(Server|ServerActive|Hostname)' /etc/zabbix/zabbix_agent2.conf

#Serveur Zabbix

timeout 3 bash -c '</dev/tcp/172.16.0.10/10050' && echo OUVERT || echo FERME
zabbix_get -s 172.16.0.10 -k agent.ping

#Windows Serveur

Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -Name fDenyTSConnections -Value 0
Enable-NetFirewallRule -DisplayGroup 'Bureau à distance'
Get-NetTCPConnection -LocalPort 3389
Get-Service TermService
Set-NetFirewallRule -DisplayGroup 'Bureau à distance' -RemoteAddress Any
