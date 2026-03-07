in this section of my report i will be focused on the identification and exploitation of diffent vulnerabilities(bugs) on a web site . my report is divided in different sections that run from the identification ,definition and exploitation of this bugs

Web app penetration testing is essentially the act of finding vulnerabilities (generally caused by poorly written code)
common ways to exploit or attack a web site include
-command injection
    this is when an attacker inserts a malicious code in a search bar of the website or  where such a code a not supposed to be written
-cross-site scripting or access attacks
    this is when an attacker inserts a javascipt code in a search bar of the website or  where such a code a not supposed to be written
-SQL injections
    here ,thes codes are written in the form of SQL queries and can can de used to access the database of the wedsite and downloead sensitive informations.
-HTLM injections
    it is pretty similar to access attacks the only difference is thatr here instead of a javascript code , and HTML code is used instead and this affects only the physical apearance of the page

in the corse of the course i learned abit more about the burp suite tool in wed penetration testing

BURP SUITE
it is a tool used to intercept and view HTTP requests and responses before they reacch the servver and user respectively
 it is a useful tool for web penetration testing since it does just intercept the requests and responses it can also be used for brute forcing

In the practical phase of my web penetration coures i used the metasploitable web page DVWA for my practice and i learned that the vulnerability of this site is divide into three  stages
hard
    here we have a more secured code and the level of filtering is considerably high and can be considered secured
meduim
    here the code is not as secured but there are still some available vulnerabilities and the code does not apropriately filters tyhe user input
low
    the security level here is the lowest and there is little or no user input filtering .

Another type of attack that can be performed on a wedsite is the shellshock attack that occurs due to patch differently processing environmental variable

Another thing i learn in this coure is the use of playloads
Playloads
these are python codes written on a machine or system for another one

to create a playload we must follow  the following steps
1) we must first know own target(s) in order to create the right playload
2) we can use command injection to download or transfer the playload to our target
3) exercute the code on the target machi e that can still be done with the help of command injection
As said above, in this practice , metasploit is used and metasploit is Linus 32 bits machine

Cross-site scripting (xss) or access attack
 it is divided in three 
 -reflected access
 -stored access
 -DOM based access
Stored  access
the attacker finds an access vulnerability, inject the Js code that the want to affect the web site and anyone that visits the web pade after that will execute the js code .
Reserved access
this is different from store access in that the code wont be saved on the server but if you coppy the link of the modified webpage and send it to some one and the clink , the code will be executed

SQL injection
usually an SQL query is written in the form
SELECT item FROM items WHERE condition 

Brute force attack
 this is not a vulnerability exploitation due to apoor code or lack of security . it is simply a methode of force our way in to a website without having the permition or right .
 this can be done with the help of Hydra which is already installed on our linux(kali machine) or with eht help of burp suite
