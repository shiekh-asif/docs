---
layout: single
type: docs
permalink: /docs//apache-hardening/
redirect_from:
  - /theme-setup/
last_modified_at: 2023-07-19
last_modified_by: Mohammad_Asif
toc: true
title: "Apache Hardening for Security Implications"
---
# Apache Hardening for Security Implications
<img alt="Apache" src="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQ7ZgLOdo0TtCe4GOWoPmYEhyXY-ax2iILAzizZbFomQl-K7PNJh-i_Q8syiwIlR0Bfw1A&usqp=CAU" width="180"/>

The web server has a crucial role in web-based applications. Since most of us leave it to the default configuration, it can leak sensitive data regarding the web server.

By applying numerous configuration tweaks we can make Apache withstand malicious attacks up to a limit. Following are some Apache web server hardening tips that you can incorporate to improve security.

 - [<strong>1. Hide Web Server Details</strong>](#1Hide-Web-Server-Details)
 - [<strong>2. Disable HTTP Trace Method</strong>](#2Disable-HTTP-Trace-Method)
 - [<strong>3. Disable Directory Listing</strong>](#3Disable-Directory-Listing)
 - [<strong>4. Click Jacking defense with X-Frame Options</strong>](#4Click-Jacking-defense-with-X-Frame-Options)
 - [<strong>5. Basic XSS Protection</strong>](#5Basic-XSS-Protection)
 - [<strong>6. Enable HttpOnly and Secure Flag</strong>](#6Enable-HttpOnly-and-Secure-Flag)
---


 <a id="1Hide-Web-Server-Details" name="1Hide-Web-Server-Details"></a>

### <strong>1. Hide Web Server Details</strong>

One of the first things to be taken care of is hiding the server version banner.

The default apache configuration will expose the server version. This information might help an attacker gain a greater understanding of the systems in use and potentially develop further attacks targeted at the specific version of the server.

Follow the below steps to hide the server version.

1. Edit your Apache server configuration file using Nano.

- For RHEL Based Systems:
```
nano /etc/httpd/conf/httpd.conf
```

- For Debian Based Systems:
```
nano /etc/apache2/conf-enabled/security.conf
```

2. Scroll down to the “ServerTokens” section where you’ll probably see multiple lines commented out (beginning with “#”) stating “ServerTokens” and different options. Change the uncommented line, likely “ServerTokens OS”, or comment out the line and create a new line to hide the Apache version and OS from HTTP headers:

```
ServerTokens Prod
```

3. The next section down should be the “ServerSignature” section. Turning this off hides the information from server-generated pages (e.g. Internal Server Error).

```
ServerSignature Off
```

4. Exit the file and save changes.
```
Ctrl + x
```

5. Restart Apache.

- For RHEL Based Systems:
```
systemctl restart httpd
```

- For Debian Based Systems:
```
systemctl restart apache2
```

 <a id="2Disable-HTTP-Trace-Method" name="2Disable-HTTP-Trace-Method"></a>

### <strong>2. Disable HTTP Trace Method</strong>

Apache versions newer than 1.3.34 and 2.0.55 (or newer) can use the variable TraceEnable to enable or disable. By default, it is enabled. TraceEnable Off causes Apache to return a 403 FORBIDDEN error to the client.

To disable HTTP TRACE/TRACK methods in Apache follow the below steps:

1. Edit your Apache server configuration file using Nano.

- For RHEL Based Systems:
```
nano /etc/httpd/conf/httpd.conf
```

- For Debian Based Systems:
```
nano /etc/apache2/conf-enabled/security.conf
```

2. Search for TraceEnable On, change this option to off

```
TraceEnable off
```
3. Exit the file and save changes.
```
Ctrl + x
```

4. Restart Apache.

- For RHEL Based Systems:
```
systemctl restart httpd
```

- For Debian Based Systems:
```
systemctl restart apache2
```

 <a id="3Disable-Directory-Listing" name="3Disable-Directory-Listing"></a>

### <strong>3. Disable Directory Listing</strong>

Directory listing is a web server function that displays the directory contents when there is no index file in a specific website directory. It is dangerous to leave this function turned on for the web server because it leads to information disclosure.

You can add -Indexes to Options directive in Apache's configuration file to fully disable directory listing, or add the same -Indexes option into Directory configuration to disable the feature per-directory.

1. Edit your Apache server configuration file using Nano.

- For RHEL Based Systems:
```
nano /etc/httpd/conf/httpd.conf
```

- For Debian Based Systems:
```
nano /etc/apache2/apache2.conf
```

2. Find the below section:

```
<Directory /var/www/>
        Options Indexes FollowSymLinks
</Directory>
```

3. Change Options Indexes FollowSymLinks to below:

```
Option -Indexes +FollowSymlinks
```

4. Exit the file and save changes.
```
Ctrl + x
```

5. Restart Apache.

- For RHEL Based Systems:
```
systemctl restart httpd
```

- For Debian Based Systems:
```
systemctl restart apache2
```

 <a id="4Click-Jacking-defense-with-X-Frame-Options" name="4Click-Jacking-defense-with-X-Frame-Options"></a>

### <strong>4. Click Jacking defense with X-Frame Options</strong>

Clickjacking is an attack that tricks a user into clicking a webpage element which is invisible or disguised as another element. This can cause users to unwillingly download malware, visit malicious web pages, provide credentials or sensitive information etc online.

X-Frame-Options allows content publishers to prevent their own content from being used in an invisible frame by attackers.

1. Edit your Apache server configuration file using Nano.

- For RHEL Based Systems:
```
nano /etc/httpd/conf/httpd.conf
```

- For Debian Based Systems:
```
nano /etc/apache2/conf-enabled/security.conf
```

2. Now add the following entry to file:

```
Header always append X-Frame-Options SAMEORIGIN
```
3. Exit the file and save changes.
```
Ctrl + x
```

4. Restart Apache.

- For RHEL Based Systems:
```
systemctl restart httpd
```

- For Debian Based Systems:
```
systemctl restart apache2
```

 <a id="5Basic-XSS-Protection" name="5Basic-XSS-Protection"></a>

### <strong>5. Basic XSS Protection</strong>

The X-XSS-Protection header is designed to enable the cross-site scripting (XSS) filter built into modern web browsers. This is usually enabled by default, but using it will enforce it. It is supported by Internet Explorer 8+, Chrome, Edge, Opera, and Safari. 

1. Edit your Apache server configuration file using Nano.

- For RHEL Based Systems:
```
nano /etc/httpd/conf/httpd.conf
```

- For Debian Based Systems:
```
nano /etc/apache2/conf-enabled/security.conf
```

The recommended configuration is to set this header to the following value, which will enable the XSS protection and instruct the browser to block the response in the event that a malicious script has been inserted from user input instead of sanitizing.

```
Header set XSS-Protection "1;mode=block"
```

3. Exit the file and save changes.
```
Ctrl + x
```

4. Restart Apache.

- For RHEL Based Systems:
```
systemctl restart httpd
```

- For Debian Based Systems:
```
systemctl restart apache2
```



  <a id="6Enable-HttpOnly-and-Secure-Flag" name="6Enable-HttpOnly-and-Secure-Flag"></a>

### <strong>6. Enable HttpOnly and Secure Flag</strong>

Cross-site scripting (XSS) attacks are often aimed at stealing session cookies.  Therefore, a method of protecting cookies from such theft was devised: a flag that tells the web browser that the cookie can only be accessed through HTTP – the HttpOnly flag.

The Secure flag is used to declare that the cookie may only be transmitted using a secure connection (SSL/HTTPS). If this cookie is set, the browser will never send the cookie if the connection is HTTP. This flag prevents cookie theft via man-in-the-middle attacks.

Note that this flag can only be set during an HTTPS connection. If it is set during an HTTP connection, the browser ignores it.

Follow the below steps:

1. Enable the required Apache modules.

```
a2enmod rewrite
```

```
a2enmod headers
```


2. Edit the Apache configuration file for the website.

```
nano /etc/apache2/apache2.conf
```
3. If your website supports only HTTP, Add the following lines at the end of the file.

```
Header edit Set-Cookie ^(.*)$ $1;HttpOnly
```

4. If your website supports only HTTPS, Add the following lines at the end of the file.

```
Header edit Set-Cookie ^(.*)$ $1;HttpOnly;Secure
```

5. Exit the file and save changes.
```
Ctrl + x
```

6. Restart Apache.

- For RHEL Based Systems:
```
systemctl restart httpd
```

- For Debian Based Systems:
```
systemctl restart apache2
```

