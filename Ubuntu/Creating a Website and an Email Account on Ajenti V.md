# Creating a Website and an Email Account on Ajenti V

```Ubuntu``` ```Email``` ```Monitoring``` ```Control Panels``` ```Arch Linux``` ```CentOS``` ```CoreOS``` ```Debian```

## Introduction


Ajenti is an open source, web-based control panel that can be used for a large variety of server management tasks. The add-on package called Ajenti V allows you to manage multiple websites from the same control panel. By now you should have Ajenti and Ajenti V installed.


In this tutorial we will setup a basic website using Ajenti V from the Ajenti control panel and create an email account on your website’s domain.


# Prerequisites


For this tutorial, you will need:


- A registered domain name that resolves to the Droplet with Ajenti and Ajenti V installed (example.com is used throughout this tutorial)
- Ajenti and Ajenti V installed from How to Install the Ajenti Control Panel and Ajenti V on Ubuntu 14.04

# Configuring your Domain


So far, https://panel.your_domain_name:8000/ opens the Ajenti control panel.  Before we can setup your website at http://your_domain_name/ and configure email addresses for your domain, there are a few DNS modifications that need to be made.


We need to add 2 records to make sure your website and email addresses work properly. Go to cloud.digitalocean.com/domains/ and click on the domain name you configured when setting up the Ajenti control panel and Ajenti V.


If you haven’t already, add an A record for the entire domain by filling out “@” in the HOSTNAME field, and selecting your Ajenti server or fililng out its IP address in the WILL DIRECT TO field.  Click the Create Record button to add this initial A record.  If panel.example.com is not yet configured for your domain, repeat the above steps again, but use “panel” instead of “@” for the hostname.


Next, select the MX record type.  In the HOSTNAME text box, enter @.  In the MAIL PROVIDERS MAIL SERVER, enter your domain name.  In the PRIORITY text box, enter 10.  Click the Create Record button to add the record.


If you are configuring the same domain that has the Ajenti control panel on it, you also need to add a TXT record. For instance, if you access Ajenti on panel.example.com, you should add this record. If you access it on panel.otherdomain.com, you do not need to add the TXT record.


To add the TXT record, select the TXT record type. In the HOSTNAME text box, enter @. In the VALUE text box, paste this in: v=spf1 a ip4:your_server_ip ~all (replacing the IP with your server’s IP). Click the Create Record button.


Your domain DNS settings should now look like this. Your specific domain will vary:





# Creating the Website Directory


In your browser, browse to https://panel.example.com/ and log into Ajenti. In the sidebar to the right, under the Web section, click Websites. The first time it may give you a notice that it is not active yet. Click the Enable button to allow Ajenti V to make the necessary config changes.


There is a section called New Website. Under that there is a Name text field. You can type anything you want to identify your website with in this field. Click the Create button, and you will notice your website is now listed under the Websites section at the top of the page. Click Manage next to your website.


Under the Website Files section, change /srv/new-website to any directory, for example /srv/example.com. Click the Set button, and then click the Create Directory button. Remember this directory. You will need to upload files to it soon.


Under the General tab, uncheck the Maintenance mode setting. Click Apply changes at the bottom of the page.


At the top of the page click on the Domains tab. Click the Add button, and type your domain name in the text field that appears. Click the Apply Changes button.


# Creating/Uploading the Website Files


Now that you have a directory for your website files, you need some files to go in it.


Under the Tools section in the sidebar, click File Manager. Click on the folder names to navigate to the directory you created for the website files such as /srv/example.com. The folder should be empty. Here you can upload whatever files and folders you would like for your static website.


For the purposes of this tutorial, we are going to make a simple “Hello world!” document on the website. Click the New File button at the top of the screen. A file named new file should appear in the folder. At the end of the line for the file, click the menu button.





In the Name field, change new file to index.html. Click Save, and open that same menu again. This time, click the Edit button to open a Notepad. The following text is an example of what you could enter to ensure your website is working. Of course, you can enter anything you’d like here instead.


index.html
```
<!DOCTYPE html>
<html>
<head>
  <title>This website is working!</title>
</head>
<body>
<h1>Hello, world!</h1>
<p>If you can read this correctly, your website is functional!</p>
</body>
</html>

```


Click Save at the top of the Notepad, and the file will go live.


Repeat these steps as needed to build your website. You can also make use of the File Manager’s upload function. Back in File Manager, towards the bottom of the screen there is a button titled Choose File. You can click that and a normal prompt to choose a file from your computer to upload will appear. From here, any files you select will be uploaded to the site.


# Browse to Your Website


Browse to your domain name such as http://example.com in your web browser. If you see the “Hello World!” page you made in the last step, everything works!





You can choose to finish now or read on to create your own email account for your own domain.


# Creating an Email Account


Now we will make an email account with your registered domain name.


Go to your Ajenti Control Panel. Under the Web section, click Mail. Click the Enable button if prompted. Under New Mailbox there are a few sections to fill out. The Address text box is whatever you want to come before the @ sign in your email address. If you enter sammy here, your email will be sammy@example.com. There is also a dropdown box to select a domain. If you have more than one website configured with Ajenti V, there will be multiple options here. Don’t put anything in the Custom domain field. After you’ve filled out this information, click + Mailbox.


As you can see, your new email address now appears under the Mailboxes section of this page. Click it, and then click the Change password link. Type a new password for your mailbox, press ENTER, and then click Apply Changes at the bottom of the screen.


Now click the Advanced tab at the top of the page. In the TLS section, check the box next to enable, then click the Generate new certificate button. After you’re done, click Apply changes at the bottom of the page. This increases security when connecting to your mailbox and increases compatibility with email clients.


To retrieve your email, you can connect with an email client (like Outlook, Thunderbird, K-9 Mail, etc.) or you can install RainLoop, a webmail program for accessing your mail in a browser in Ajenti.


To connect to this mailbox in an email client, the following information should be useful:


```
Username: user@example.com
Password: your_mailbox_password
IMAP Server: panel.example.com
IMAP Port: 143
IMAP Encryption: STARTTLS (Accept all certificates)
SMTP Server: panel.example.com
SMTP Port: 25
SMTP Encryption: None

```


# Conclusion


You should now have a working email account and website for your domain name. They were both created within the Ajenti V Control Panel and can both be modified with the same control panel.


Check out Installing the RainLoop Email Client on Ajenti V if you want to learn how to install a web-based email client on your server.


