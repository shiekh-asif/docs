# How to enable MOD-SECURITY for Apache2

> This artical describes on how to implement Mod-Security on Apache 2 with OWASp Rules.


## About Mod-Security and OWASP Rules

### Mod-Security:

ModSecurity is an open-source web-based firewall application (or WAF) supported by different web servers: Apache, Nginx and IIS. The module is configured to protect web applications from various attacks. ModSecurity supports flexible rule engine to perform both simple and complex operations.

### OWASP Rules:

The OWASP ModSecurity Core Rule Set (CRS) is a set of generic attack detection rules for use with ModSecurity or compatible web application firewalls. The CRS aims to protect web applications from a wide range of attacks, including the OWASP Top Ten, with a minimum of false alerts. The CRS provides protection against many common attack categories, including SQL Injection, Cross Site Scripting, Local File Inclusion, etc.

## Install Mod-Security on Ubuntu | Debian for Apache2:

1. To install Mod-Security we need to update and install the package to the from the APT repositiory to do the same follow the below commands.

`sudo apt-get update`

`sudo apt-get install libapache2-mod-security2`

2. After installing the mod-security we need to enable the mod-security module in apache to do the same run the below command.

`a2enmod security2`

3. Then restart apache2 by running the below command.

`systemctl restart apache2`

4. By default, ModSecurity is only configured to detect and log suspicious activity. We need to go an extra step and configure it to not only detect but also block suspicious activity.

5. Copy, the default ModSecurity configuration file – modsecurity.conf-recommended – to a new file as provided in the command below.

`sudo cp /etc/modsecurity/modsecurity.conf-recommended /etc/modsecurity/modsecurity.conf`

6. Using your preferred text editor, open the file.

`sudo nano /etc/modsecurity/modsecurity.conf`

- Locate the line:

`SecRuleEngine DetectionOnly`

- And change it to bellow:

`SecRuleEngine On`

7. To make the changes effect, restart apache2.

`systemctl restart apache2`

Mod-Security is installed in the server by now we need to install the OWASP rules now.

## Adding OWASP Rules to the Mod-Security:

1. The next step is to download the latest OWASP ModSecurity Core Rule Set (CRS) from the GitHub page. We are using the verion 3.3.0 of the owasp rules are the latest rules are facing some syntax error.

> The below is to download the latest version of OWASP rules which we are not using at the moment:
> `git clone https://github.com/coreruleset/coreruleset.git`


3. Use the below command to download the required version of OWASP Rules:

`wget https://github.com/coreruleset/coreruleset/archive/v3.3.0.zip`

4. The unzip the downloaded and go inside the directory by runing the below command:

`unzip v3.3.0.zip`

`cd coreruleset-3.3.0`

5. Now we need to move the crs-setup configuration file from the coreruleset folder to the mod security foleder to do so run the below command.

`sudo mv crs-setup.conf.example /etc/modsecurity/crs-setup.conf`

6. Now we need to move the rules from the coreruleset folder to the modsecurity to enable the rules.

`sudo mv rules/ /etc/modsecurity/`

7. We need to show apache to use the rules by making sure we have this lines in the security2.conf module configuration file to do so we need to run the below command.

`sudo nano /etc/apache2/mods-enabled/security2.conf`

 Ensure we have the below lines in the file: 

`	IncludeOptional /etc/modsecurity/*.conf`

`	Include /etc/modsecurity/rules/*.conf`

Find the following line, which loads the default CRS files.

`IncludeOptional /usr/share/modsecurity-crs/*.load`

Change it to the following, so the latest OWASP CRS will be used.

`IncludeOptional /etc/apache2/modsecurity-crs/coreruleset-3.3.0/crs-setup.conf`

`IncludeOptional /etc/apache2/modsecurity-crs/coreruleset-3.3.0/rules/*.conf`

8. Now restart the apache to make the rules effect.

`systemctl restart apache2`

9. Then we need to enable the modsecurity for the website by adding the below lines at the bottom of the file above `</virtualhost>`.

`SecRuleEngine On`

`SecRule ARGS:testparam "@contains test" "id:254,deny,status:403,msg:'Test Successful'"`

10. After adding the above lines to the virtualhost file we need to restart apache2 to do so run the bellow command.

`sudo systemctl restart apache2`

11. We can check the mode security is enabled by browsing the below in the browser it will give 403 forbidden error.

`https://server-ipordomain/?testparam=test`
and checking the error log both in the server and the browser it will throw 403 forbidden error in browser and (test successful) in the apache error log.

12. Now the mod security is enabled with the OWASp rules we need to make few changes to the rules so that our application/website will work without any issues.

13. We need to edit this file to add the methods to PUT DELETE etc.. (based on the requirement), to edit the file run this command.

`nano /etc/modsecurity/crs-setup.conf`
find the below lines and uncomment them and add the lines as below.
```
#SecAction \
# "id:900200,\
#  phase:1,\
#  nolog,\
#  pass,\
#  t:none,\
#  setvar:'tx.allowed_methods=GET HEAD POST OPTIONS'"
```
after uncommneting the above lines and add the DELETE PUT PATCH as below.
```
SecAction \
 "id:900200,\
  phase:1,\
  nolog,\
  pass,\
  t:none,\
  setvar:'tx.allowed_methods=GET HEAD POST OPTIONS DELETE PUT PATCH'"
```

14. Once the above are done the faveo will work without any issues we need to restart the apache2 so the changes to be effect.

`systemctl restart apache2`


> MOD-SECURITY for apache2 is configured successfull and the OWASP rules are in effect.










