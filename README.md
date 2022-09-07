# Log4j-workshop

Welcome to CircleCityCon and today's training:
- Log4j Vulnerability: Emulation and Detection

The entirety of the instructions will be available in this GitHub Repo. To save a copy for reference:
`git clone https://github.com/Oofles/Log4j-workshop.git`

## Choose Your Own Adventure
Please choose one of the available options!

### **Option 1: Pluralsight Cloud Environment**
With this option, everything is setup and configured for you! This environment also contains two machines so you can see the attack working across endpoints.

1. Create a free (No credit card required) Pluralsight Account via this link: [Click Here!](http://track.pluralsight.com/MzA2LURVUC03NDUAAAGCiepSmIeMhu-uAa7Z6CLy83pQ_v_Q4ZOIF4Wvgd-eCMQULN0uqw6O_gBcD3TtEhYSnSAncSM=)

2. Navigate to the Lab: [Log4j Vulnerability Lab: Emulation and Detection](https://app.pluralsight.com/labs/detail/1874f406-cb9a-44a0-841e-c171ce0aebcb/toc)


### **Option 2: Local Setup**
With this option, you bravely choose to set this up on your own laptop!

Prerequisites:
- Docker 
- ~4GB of free RAM
- tcpdump
- Suricata

1. Pull down the containers:
```
docker pull oofles/log4shell-ldap-server
docker pull oofles/vuln-log4j-app
```

2. Start the vulnerable web application (This will be hosted on port 8080 by default)
```
sudo docker run -d --name vulnapp2 --network host oofles/vuln-log4j-app
```

3. All other instructions will be very similar, with minor modifications for your file system, IPs, and ports.


## Modules
The instructions for the various modules are below:

- [1-Setup Network Detections](1-Setup_Network_Detections.md)
- [2-Setup Malicious LDAP Server and POC](2-Setup_Malicious_LDAP_Server_and_POC.md)
- [3-Upgrade to a Reverse Shell](3-Upgrade_to_a_Reverse_Shell.md)
- [4-Analysis and Detection](4-Analysis_and_Detection.md)

