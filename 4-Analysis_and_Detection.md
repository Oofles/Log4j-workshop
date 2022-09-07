# 4 - Analysis and Detection

In this challenge you will be taking a look back at Suricata to see if it alerted on anything. You'll also be diving into the PCAP with tcpdump to inspect the traffic seen.

1.  Start by connecting to the **Ubuntu Defender Endpoint.**

2.  At this point you can stop Suricata and tcpdump from running:
	- `screen -r suricata`
	- `CTRL + C` to stop the application
	- `CTRL + A`, let go, then press `D` to detach
	- Repeat for tcpdump.

3.  Suricata was set to log in the **/home/ubuntu/lab/** directory. List out the contents to see the logs that are created:

`ls ~/lab`

4.  The **fast.log** contains short descriptions with a single alert per line. Take a look at what's in this file:

`less -S ~/lab/fast.log`

**Note:** When using `less`, you can read through a file with the arrow keys. When you are finished, press `q` to quit.

**Analysis:** Multiple alerts have triggered! General information available in these alerts include the IPs and ports involved. In a typical enterprise environment, an IDS (like Suricata) would be configured to forward these alerts to a Security Information and Event Management (SIEM) system. You can also configure your system to email or send additional messages when certain alerts are triggered.

5.  We don't have a SIEM configured in this environment, and you may need to dig through Suricata logs on the command-line at some point. Let's take a look at how to do that and start by looking at the **eve.json** file:

`cat ~/lab/eve.json | jq . | less`

**Note:** The Extensible Event Format (nicknamed EVE) log contains all the network information, alerts, metadata, and protocol specific logs that Suricata generates. Even though Suricata is typically used as an IDS, it can generate a lot of useful information gathered from the network traffic.

**Note:** `jq` is a Linux utility that assists with reading through JSON documents on the command line. For more information see: [https://stedolan.github.io/jq/](https://stedolan.github.io/jq/)﻿

6.  Since there is a massive amount of information in this log, and we are currently only interested in the alerts, you can use `jq` to filter this information down:

`cat ~/lab/eve.json | jq 'select(.event_type == "alert")' | less` 

**Analysis:** Looking at the alerts in the EVE log, you can see a lot more information. Because these attacks happened over HTTP, Suricata will also create entries for the protocol metadata like hostname, URL, user agent, method, and status. This is extremely helpful for initial analysis and collecting artifacts for incident response.

7.  If you wanted to get all the information from a particular conversation (or flow), you can use the **flow_id** field seen in the alert:

`cat ~/lab/eve.json | jq 'select(.flow_id == 769469750893595)' | less`

**Note:** The **flow_id** is a unique value and generated at runtime. You will need to look at the value in your own environment for this filter to work properly.

1.  One final thing you may want to do is extract individual values out of a field. As an example, filter for all the alerts and generate a count of source IPs.

`cat ~/lab/eve.json | jq 'select(.event_type == "alert") | .src_ip' | sort | uniq -c | sort -n`

**Note:** This filter process can be repeated for any of the available fields you'd like more information on.

**Analysis:** Even though all the alerts come from a single IP in our lab environment, this type of filtering is extremely helpful when performing analysis on a live network where you may have thousands of alerts.

9.  Using the information gathered from the alerts, now you can pivot to the PCAP file and use tcpdump to gain some additional information by filtering on the attacker's IP:

`tcpdump -nn -r ~/analysis.pcap 'src host 172.31.24.10' | less -S`

**Analysis:** In a typical incident response scenario, you'd need to dig through the conversations to understand what actually happened. Because you have the benefit of having run the attack yourself, you can skip some of the guess work and analyze the flows individually.

10.  Start by taking a look at the requests to the vulnerable application:

`tcpdump -nn -r ~/analysis.pcap 'tcp port 8080' -vvv | less -S`

**Note:** The `-v` option in tcpdump prints out information in increasing levels of verbosity. With this you can see useful information like the HTTP headers.

**Analysis:** Some Log4j attacks take advantage of fields like the user agent or URL to submit the malicious queries. Since you submitted the malicious payload utilizing the **X-Api-Version** field, you can filter onto that with this approach:

`tcpdump -nn -r ~/analysis.pcap 'tcp port 8080' -vvv | grep "X-Api-Version" | sort | uniq -c`

**Analysis:** You may also use this method to perform a broad search for **jndi**, **ldap**, or **rmi** lookups in network traffic. This is not an efficient approach and may take a long time depending on the amount of network traffic you have. Simple obfuscation methods can hide from this as well. An example:

`tcpdump -nn -r ~/analysis.pcap -vvv | grep jndi`

11.  Next, move onto the LDAP requests over port 1389:

`tcpdump -nn -r ~/analysis.pcap 'tcp port 1389' -vvv -X | less -S`

**Note:** Using tcpdump, `**-X**` prints out the complete packet contents in ASCII and hex.

**Analysis:** You will have to scroll through the packets but will be able to see a couple artifacts. First, the Base64 command sent from the victim back to the malicious server for redirection. Then, the response back with a redirection to port **8888** and calls to specific java classes.

12.  With that, check port **8888**:

`tcpdump -nn -r ~/analysis.pcap 'tcp port 8888' -vvv -X | less -S`

**Note:** Scrolling down with `<Space>`, `<Return>`, or arrow keys will show more information.

**Analysis:** First you will see the request to the malicious LDAP server. Eventually, there is a larger packet returned from the server where you can see Java code being initialized. If you scroll down near the bottom, you can see the command sent in plain text.

13.  One last thing to look at! Filter on port **7777** for the reverse shell:

`tcpdump -nn -r ~/analysis.pcap 'tcp port 7777' -vvv -X | less -S`

**Analysis:** Depending on what commands you ran, you will be able to see the commands sent and the information received, since this is a non-encrypted Command and Control (C2) channel. This type of packet analysis may be easier with a tool like Wireshark, but it's very helpful to have these skills in case you are in a limited environment or need quick information in the middle of an incident response.

There are many different detection methods depending on the configuration of your environment and what software you may have installed. Please check with your software vendors to see if they have additional resources.