---
layout: post
title: Command Injection
author: Eduardo Souza
date: '2020-07-23 17:58:23 -0300'
category:
        - security
summary: How does it work and how can we prevent it
thumbnail: /assets/img/posts/command_injection.jpg
---

By using a tool like Netcat utility to read from a network connection using TCP and to output the results of the exploited Command Injection vulnerability to a consolean attacker can checks his CLI window and sees there the content of the _C:\Windows\windows.ini_ file from the server.

With this vulnerability, **almost any arbitrary operating system command can be executed** on the server.
The most effective way to prevent Command Injection vulnerabilities is to avoid calling OS commands from the application code.
If the execution of OS commands cannot be avoided, don't use user-supplied input to run them.
If it is not possible to avoid user-supplied input in calling out to OS commands, then strong input validation must be implemented. It must be based on a allowlist of allowed characters. Don't use banlists because they tend to be bypassed by a skilled attacker.
> Please note that even if you allowlist only alphanumeric symbols, and the attacker fails to break the syntax of the existing command and inject his malicious command, he could still be able to add an argument to the existing command that would change the result of its execution.
 
A defense-in-depth approach also recommends to carefully set permissions for the application and its components to prevent OS commands execution.
 
A attacker will can also try to trigger the resource exhaustion denial of service.
 
Resource exhaustion can affect the following areas of functionality:
 
* Storage:
  * With the storage depletion attack, server disk space runs out, thus preventing the server from functioning properly.
 
* Connections:
  * When the attacker simultaneously uploads the big number of files and sends each packet with a considerable delay thus keeping all the connections alive, it may lead to the depletion of the number of connections. Generally, an attacker clogs the communication pipeline of the application and therefore not only prevents further file uploads but causes Denial of Service of the application as a whole.
 
 * Resource exhaustion could lead to the **Denial of Service** of the whole application or a separate function of that application. It happens when a server consumes 100% of the available resource (CPU, memory, disk space, network bandwidth, etc.) and is not able to process any further requests. File upload could lead to the resource exhaustion when the file storage is misconfigured and necessary security controls are not implemented at the application and OS levels. 
 
Resource exhaustion is not the only bad thing that could happen during the unrestricted file upload. It may also lead to the following attacks depending on the context of how the uploaded file is used:
 
 * The code in uploaded files could be executed. For example, instead of a picture, an attacker could load a Web shell and run OS commands on the server with the Web server privileges.
 
 * Existing critical files could be overwritten. In turn, overwriting the access control or configuration files could lead to opening new security breaches in the application.
 
 * Sensitive information could get exposed (in case the ACL is misconfigured and allows horizontal or vertical privilege escalation).
 
 * Malicious or unauthorized data could be distributed. For example, when some malicious files are added to the file share and are accessible to application users as is, without file content check.
 
Given a virtually unlimited amount of users with a virtually unlimited amount of time and bandwidth, any system's resource can be eventually exhausted, but a defense-in-depth approach can make exhaustion significantly harder. The following security controls should form a multi-layered defense mechanism against storage depletion:
 
 * File size check. Uploading large files could cause a depletion of disk space. The maximum size of the file should be set according to the business requirements and communicated to users.
 
 * File name check. A long filename could lead to unexpected errors or memory issues. If the business logic of the application allows, the names of the uploaded files should be restricted in length, sanitized using a whitelist of allowed characters, or changed to random strings.
 
 * Restriction on the amount or total size of uploaded files by a single user. This security control could help against a distributed file upload attack (if the creation of a new user is not trivial and an attacker cannot create thousands of users) and could drastically decrease the risk of disk space exhaustion. Make sure to align the restrictions with the application business requirements and to avoid breaking the user experience of non-malicious users.
 
 * File upload or parsing library should be up to date. If it is not updated regularly, it could contain a vulnerability that leads to denial of service. For example, when a malicious file is loaded using a vulnerable library, it could send the library to an infinite loop of parsing the file.
 
 * Resource monitoring. Depletable or finite server resources should be monitored by the application support or operations team: the overall amount of disk space left, the disk space allocated for storing the uploaded files, the number of active and long-lasting connections to the web server, etc. Once the operations team is notified of suspicious activity, they can get involved to manually prevent DoS.
 
 * Scalable storage, such as via a private cloud. If storage is scalable, it could expand when required, helping both in cases of intentional exhaustion and also during abnormally high activity spikes.
 
