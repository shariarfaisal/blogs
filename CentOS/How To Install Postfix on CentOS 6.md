# How To Install Postfix on CentOS 6

```Email``` ```CentOS```










# Status: Deprecated


This article covers a version of CentOS that is no longer supported. If you are currently operating a server running CentOS 6, we highly recommend upgrading or migrating to a supported version of CentOS.


Reason:
CentOS 6 reached end of life (EOL) on November 30th, 2020 and no longer receives security patches or updates. For this reason, this guide is no longer maintained.


See Instead:
This guide might still be useful as a reference, but may not work on other CentOS releases. If available, we strongly recommend using a guide written for the version of CentOS you are using.


## About Postfix


Postfix is free open source Mail Transfer Agent which works to route and deliver email. Cyrus is a server that helps organize the mail itself.


# Step One — Install Postfix and Cyrus


The first thing to do is install postfix and Cyrus on your virtual private server and the easiest way to do this is through the yum installer. 



```
sudo yum install postfix
```


```
sudo yum install cyrus-sasl
sudo yum install cyrus-imapd
```


Say Yes to the prompt each time it asks. Once all components have downloaded, you will have postfix and cyrus installed.


# Step Two — Configure Postfix


We are going to configure both programs separately.


First, open up the Postfix’s main configuration file.


```
sudo vi /etc/postfix/main.cf
```


The postfix configuration file is very handy and detailed, providing almost all of the information needed to get the program up and running on your VPS. Unfortunately this also makes for a very long file. 


The suggested code below is, in most regards, simply a shortened, and correctly uncommented version of what is in the file already. For a quick set up that will provide you with all of the needed configs to set up postfix, copy and paste the information below over Postfix's current configuration. Be careful to correct the domain names under myhostname and my domain.


Replace the example.com in the myhostname line with a DNS approved domain name. Be sure that the phrase is still mail.yourdomainnamehere


Replace the example.com in the mydomain line with the correct domain name.


```
soft_bounce             = no
queue_directory         = /var/spool/postfix
command_directory       = /usr/sbin
daemon_directory        = /usr/libexec/postfix
mail_owner              = postfix

# The default_privs parameter specifies the default rights used by
# the local delivery agent for delivery to external file or command.
# These rights are used in the absence of a recipient user context.
# DO NOT SPECIFY A PRIVILEGED USER OR THE POSTFIX OWNER.
#
#default_privs = nobody

myhostname              = mail.example.com 
mydomain                = example.com

mydestination           = $myhostname, localhost
unknown_local_recipient_reject_code = 550

mynetworks_style        = host
mailbox_transport       = lmtp:unix:/var/lib/imap/socket/lmtp
local_destination_recipient_limit       = 300
local_destination_concurrency_limit     = 5
recipient_delimiter=+

virtual_alias_maps      = hash:/etc/postfix/virtual

header_checks           = regexp:/etc/postfix/header_checks
mime_header_checks      = pcre:/etc/postfix/body_checks
smtpd_banner            = $myhostname

debug_peer_level        = 2
debugger_command =
         PATH=/bin:/usr/bin:/usr/bin:/usr/X11R6/bin
         xxgdb $daemon_directory/$process_name $process_id & sleep 5

sendmail_path           = /usr/sbin/sendmail.postfix
newaliases_path         = /usr/bin/newaliases.postfix
mailq_path              = /usr/bin/mailq.postfix
setgid_group            = postdrop
html_directory          = no
manpage_directory       = /usr/share/man
sample_directory        = /usr/share/doc/postfix-2.3.3/samples
readme_directory        = /usr/share/doc/postfix-2.3.3/README_FILES

smtpd_sasl_auth_enable          = yes
smtpd_sasl_application_name     = smtpd
smtpd_recipient_restrictions    = permit_sasl_authenticated,
                                  permit_mynetworks,
                                  reject_unauth_destination,
                                  reject_invalid_hostname,
                                  reject_non_fqdn_hostname,
                                  reject_non_fqdn_sender,
                                  reject_non_fqdn_recipient,
                                  reject_unknown_sender_domain,
                                  reject_unknown_recipient_domain,
                                  reject_unauth_pipelining,
                                  reject_rbl_client zen.spamhaus.org,
                                  reject_rbl_client bl.spamcop.net,
                                  reject_rbl_client dnsbl.njabl.org,
                                  reject_rbl_client dnsbl.sorbs.net,
                                  permit

smtpd_sasl_security_options     = noanonymous
smtpd_sasl_local_domain         = 
broken_sasl_auth_clients        = yes

smtpd_helo_required             = yes 
```


# Step Three — Finalize Postfix


After pasting in the proper configs, we are almost finished setting up postfix on our virtual server.


To forestall any errors, we need to execute two more steps


In the config we included virtual aliases with the line, virtual_alias_maps      = hash:/etc/postfix/virtual; now we have to set up that database.


Open that file:


```
sudo vi /etc/postfix/virtual
```


Delete all the text within the file and then add the following single line, substituting an actual username for user, and the correct domain for example.com:


```
user@example.com   user\@example.com
```


Save and exit.


Follow up by typing in this into terminal


```
postmap /etc/postfix/virtual 
```


This will turn the virtual file into a lookup table, creating the database required for postfix to work.


Finally conclude by using this command, which will create the new file that postfix expects before sending anything out.


```
touch /etc/postfix/body_checks 
```


Once all that is completed we can finish up by configuring Cyrus.


# Step Four — Configure Cyrus


The first step is to add the smtpd.conf file, which defines the authentication for Postfix/SASL, to the SASL directory:


```
sudo vi /etc/sasl2/smtpd.conf
```


Go ahead and copy and paste the following text in:


```
pwcheck_method: auxprop
auxprop_plugin: sasldb
mech_list: PLAIN LOGIN CRAM-MD5 DIGEST-MD5 
```


Save and Exit.


Next, we need to configure the Cyrus file:


```
sudo vi /etc/imapd.conf
```


Delete what is in the file currently, and paste the configurations below into the file, changing the default domain and server name to match your personal domain name.


```
virtdomains:		userid
defaultdomain:		example.com
servername:		example.com
configdirectory:	/var/lib/imap
partition-default:	/var/spool/imap
admins:			cyrus
sievedir:		/var/lib/imap/sieve
sendmail:		/usr/sbin/sendmail.postfix
hashimapspool:		true
allowanonymouslogin:	no
allowplaintext:		yes
sasl_pwcheck_method:	auxprop
sasl_mech_list:		CRAM-MD5 DIGEST-MD5 PLAIN
tls_cert_file:		/etc/pki/cyrus-imapd/cyrus-imapd.pem
tls_key_file:		/etc/pki/cyrus-imapd/cyrus-imapd.pem
tls_ca_file:		/etc/pki/tls/certs/ca-bundle.crt

autocreatequota:		-1
createonpost:			yes
autocreateinboxfolders:		spam
autosubscribeinboxfolders:	spam 
```


Save and Exit.


# Step Five — Install a Mail Client


Success! You have installed Postfix and Cyrus on your VPS. However, both of these programs relate to handling email rather than sending it. We can quickly install a method of sending messages from the command line.


There are a variety of clients we can use—here we will connect with MailX


```
yum install mailx
```


After you agree to the prompt, mailx will finish up installing.


Then, to send emails, type this command into terminal, substituting in the email that you are looking to send your message to.


```
mail user@example.org
```


Terminal will ask for a subject line. Type one in, then press enter. On the subsequent lines you can type your message. It will only be sent when you press enter, and type in a period.


Your letter will look something like this:


```
[root@demoserver ~]# mail user@example.org
Subject: Hello
This is a test message.
Regards,

.
EOT 
```


Congratulations—now you have postfix installed and email running. You are all set to use your virtual private server to send email.


By Etel Sverdlov
