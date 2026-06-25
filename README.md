# Setup Ubuntun Server

### Network

```bash
cat >/etc/netplan/50-cloud-init.yaml <<'EOF'
network:
  version: 2
  ethernets:
    eth0:
      match:
        macaddress: "7a:d0:ad:c3:5d:74"
      addresses:
        - "159.195.139.249/22"
        - "2a0a:4cc0:c1:a0b3:78d0:adff:fec3:5d74/64"
      nameservers:
        addresses:
          - 1.1.1.1
          - 1.0.0.1
          - 8.8.8.8
          - 8.8.4.4
      routes:
        - to: default
          via: "159.195.136.1"
        - to: default
          via: "fe80::1"
          on-link: true
EOF
```

```bash
netplan try
netplan apply

resolvectl dns eth0
ping github.com
wget https://raw.githubusercontent.com/buildplan/du_setup/refs/heads/main/du_setup.sh
```

### Hardening

- [Debian & Ubuntu Server Setup & Hardening Script](https://github.com/buildplan/du_setup)

```bash
wget https://raw.githubusercontent.com/buildplan/du_setup/refs/heads/main/du_setup.sh
chmod +x du_setup.sh
```

```bash
# Download the official checksum file
wget https://raw.githubusercontent.com/buildplan/du_setup/refs/heads/main/du_setup.sh.sha256

# Run the check (it should output: du_setup.sh: OK)
sha256sum -c du_setup.sh.sha256
```

```bash
whoami
hostname
./du_setup
```

```bash
Enter server hostname: v2202606227753474632
Enter a 'pretty' hostname (optional): example-server
Enter custom SSH port (1024-65535) [22]: 22
...
2) CrowdSec (Modern, collaborative reputation database, highly recommended)
```

#### Crowdsec

- [Crowdsec Collections](https://app.crowdsec.net/hub/collections)

```bash
Enter collection name: crowdsecurity/linux
→ No additional log file required

Enter collection name: crowdsecurity/iptables
Enter log file path: /var/log/ufw.log
Enter label type: iptables

Enter collection name: crowdsecurity/http-cve
→ No additional log file required

Enter collection name: crowdsecurity/base-http-scenarios
→ No additional log file required

Enter collection name: crowdsecurity/traefik
# Only if Traefik access logs are written to a file
Enter log file path: /var/log/traefik/access.log
Enter label type: traefik

→ DONE!
```

### Coolify

- [Coolify Installtion - Docs](https://coolify.io/docs/get-started/installation)

```bash
curl -fsSL https://cdn.coollabs.io/coolify/install.sh | sudo bash
```

```bash
# Prüfen, ob CrowdSec Port 8080 belegt
ss -tulpn | grep :8080

# Aktuellen CrowdSec API-Port anzeigen
grep -n "listen_uri" /etc/crowdsec/config.yaml

# CrowdSec Local API ändern:
# listen_uri: 127.0.0.1:8080
# ->
# listen_uri: 127.0.0.1:8081
vi /etc/crowdsec/config.yaml

# Firewall Bouncer ändern:
# api_url: http://127.0.0.1:8080/
# ->
# api_url: http://127.0.0.1:8081/
vi /etc/crowdsec/bouncers/crowdsec-firewall-bouncer.yaml

# Local API Credentials ändern:
# url: http://127.0.0.1:8080
# ->
# url: http://127.0.0.1:8081
vi /etc/crowdsec/local_api_credentials.yaml

# Prüfen, dass keine Referenz auf 8080 mehr existiert
grep -R "127.0.0.1:8080" /etc/crowdsec /etc/crowdsec-firewall-bouncer 2>/dev/null

# Prüfen, dass alle Referenzen auf 8081 zeigen
grep -R "127.0.0.1:8081" /etc/crowdsec /etc/crowdsec-firewall-bouncer 2>/dev/null

# CrowdSec neu starten
systemctl restart crowdsec

# Firewall Bouncer neu starten
systemctl restart crowdsec-firewall-bouncer

# Prüfen, dass CrowdSec jetzt auf 8081 lauscht
ss -tulpn | grep 808

# Docker-Container prüfen
docker ps

# Alle Container inkl. gestoppter anzeigen
docker ps -a
```

- Danach in Coolify Interface den Proxy Starten (Server > Localhost)



