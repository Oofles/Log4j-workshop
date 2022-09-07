# 1 - Setup Network Detections

In this environment, there are two Ubuntu endpoints. One is for the attacker and the other to host the vulnerable applications and provide an environment for detection. This first challenge will involve setting up the network detections and packet capture.

**Important Note**: We are building a custom environment just for you! This may take a few minutes depending on its complexity.

-   In the terminal environment, typing `status` will return an overview on applications installing and/or lab files being downloaded. Once this returns **SYSTEM COMPLETE**, your environment will be ready for full exploration! 

1.  Start by connecting to the **Ubuntu Defender Endpoint**

2.  Create a new terminal session for Suricata:

    `screen -S suricata`
    
    **Note:** Multiple terminals will be required to manage the various tasks. Since we are presenting the terminal environment through a browser, the Linux tool `screen` will assist with this.
    -   `screen -S [name]` = Creates a new terminal session and names it
    -   `CTRL + A`, let go, and then press `D` = Disconnects from a terminal session
    -   `screen -r [name]` = Reattaches to an available terminal session
    -   `screen -ls` = Lists any available screen terminal sessions and their status

3.  Start Suricata with the available log4j rules:
    
    `sudo suricata -S ~/lab/emerging-threats-log4j.rules -l ~/lab -i eth0`
    
    **Note:** There may be a "Warning" that appears stating the flowbit 'ET.RMIRequest' is checked but not set. This is expected and will not affect your detections. 
    
4.  Detach from that session by using the button key combinations of `CTRL + A`, let go, and then press `D`.
    
5.  Create a new terminal session for tcpdump:
    
    `screen -S tcpdump`
    
6.  Start tcpdump:
    
    `sudo tcpdump -nn 'not port 22 and not port 443' -w analysis.pcap`
    
    **Note:** To reduce the noise, we are filtering out port 22 (your SSH sessions) and port 443 (the AWS instances generate a lot of automatic HTTPS traffic). 
    
7.  Detach from that session by using the button key combinations of `CTRL + A`, let go, and then press `D`.
    
8.  At this point you should have two screen sessions. You can check this with:
 `screen -ls`
   
9.  The vulnerable application has been started automatically in a local Docker container. Test to make sure the application is responding:
    
    `curl localhost:8080`
    
    **Note:** The application will respond with an error and 400 status code. This is expected and means the application is responding successfully.

10.  The Proof of Concept (POC) test will involve creating a new file in the application's **/tmp/** directory. Check its current files and take note of what you see there:

`sudo docker exec vulnapp2 ls /tmp` 

Suricata is an open-source Intrusion Detection System (IDS) we will be using to detect Log4j exploits inside network traffic. If you are interested in learning more about Suricata, check out: [Network Security Monitoring with Suricata](https://app.pluralsight.com/library/courses/network-security-monitoring-suricata/table-of-contents).

Congratulations! It may not seem like much just yet, but you've setup the infrastructure for detection and tested to ensure the vulnerable application is operating properly. We'll come back here, but for now let's move on to the attacker.