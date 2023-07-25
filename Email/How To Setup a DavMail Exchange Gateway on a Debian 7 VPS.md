# How To Setup a DavMail Exchange Gateway on a Debian 7 VPS

```Security``` ```Email``` ```Java``` ```Debian```

## Introduction


If your workplace or school uses Microsoft Exchange for E-mail, you may wish to access your Exchange E-mail account from E-mail clients that do not support the Exchange protocol.


DavMail provides a solution, translating Microsoft Exchange to open protocols like POP, IMAP, SMTP, Caldav, Carddav, and LDAP.


# Installation


Davmail requires some extra dependencies to work properly. Install them with apt:


```
sudo apt-get install default-jre libswt-gtk-3-java libswt-cairo-gtk-3-jni

```


The DavMail project makes a Debian package available on their website through [SourceForge (http://sourceforge.net/projects/davmail/files/davmail/).


Download the latest Debian package with wget:


```
wget http://sourceforge.net/projects/davmail/files/davmail/4.4.1/davmail_4.4.1-2225-1_all.deb

```


Then, install DavMail with dpkg:


```
sudo dpkg -i davmail_4.4.1-2225-1_all.deb

```


# Basic Configuration


DavMail’s configuration file does not exist by default. Create one with your favorite text editor:


```
sudo nano /etc/davmail.properties

```


Set DavMail to server mode so it doesn’t require X11:


```
davmail.server=true

```


Enable remote mode and set the bind address to your droplet’s IP address or set it blank:


```
davmail.allowRemote=true
davmail.bindAddress=

```


Set davmail.url to your Outlook Web App/Outlook Web Access URL, which usually ends in /owa:


```
davmail.url=https://yourcompany.com/owa

```


Set your connection mode:


```
davmail.enableEws=auto

```


Set your port options:


```
davmail.imapPort=993
davmail.smtpPort=465
davmail.ldapPort=636
davmail.popPort=995
davmail.caldavPort=8443

```


Save and close the configuration file.


# Create A SSL Certificate


In order to enable SSL encryption, you will need a SSL certificate and SSL private key in the PEM format. If you have purchased a certificate from a Certificate Authority, then you should already have your certificate and key. If so, continue to the Configuring SSL section below. Otherwise, you can generate a self-signed certificate by following these steps.


Generate a RSA key with OpenSSL:


```
sudo openssl genrsa -out /usr/lib/ssl/private/davmail.key 2048

```


Make sure the key is owned by root and permissions are set properly:


```
sudo chown root:root /usr/lib/ssl/private/davmail.key
sudo chmod 600 /usr/lib/ssl/private/davmail.key

```


Now, create a certificate signing request:


```
sudo openssl req -new -key /usr/lib/ssl/private/davmail.key -out /usr/lib/ssl/certs/davmail.csr

```


OpenSSL will now ask you several questions. The only important field is Common Name, which should be set to the domain name or IP address of your droplet which will be accessed by your E-mail clients (e.g. davmail.mydomain.com or 123.123.123.123). The other fields can be left at their defaults by just pressing enter or can be filled in with anything:


```
You are about to be asked to enter information that will be incorporated into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
    -----
Country Name (2 letter code) [XX]:US
State or Province Name (full name) []:New York
Locality Name (eg, city) [Default City]:New York City
Organization Name (eg, company) [Default Company Ltd]:Lolcats United
Organizational Unit Name (eg, section) []:Keyboard Cat Department
Common Name (eg, your name or your server's hostname) []:mydomain.com
Email Address []:me@mydomain.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:

```


Sign the certificate request using your private key, setting the expiration date with the -days argument:


```
sudo openssl x509 -req -signkey /usr/lib/ssl/private/davmail.key -in /usr/lib/ssl/certs/davmail.csr -out /usr/lib/ssl/certs/davmail.crt -days 365

```


With the settings above, the certificate will expire in 365 days (a year).


You now have your own SSL certificate!


# Configuring SSL


Now that you have your SSL certificate, you will have to convert it into a format DavMail understands. The following examples will use the key and certificate we generated above. If you purchased a certificate from a Certificate Authority, then use those files in place of davmail.key and davmail.crt.


Start by combining your certificate and key file with cat:


```
sudo -s cat /usr/lib/ssl/private/davmail.key /usr/lib/ssl/certs/davmail.crt > /usr/lib/ssl/certs/davmail.pem
exit

```


Once again, set permissions so only root can access the key file:


```
sudo chown root:root /usr/lib/ssl/certs/davmail.pem
sudo chmod 600 /usr/lib/ssl/certs/davmail.pem

```


Now convert your combined key and certificate to a pkcs12 file:


```
sudo openssl pkcs12 -export -in /usr/lib/ssl/certs/davmail.pem -out /usr/lib/ssl/certs/davmail.p12 -name “davmail”

```


You will be prompted to enter an export password. This can not be blank!


You must set a password or DavMail will not work properly.


Set permissions:


```
sudo chown root:root /usr/lib/ssl/certs/davmail.pem
sudo chmod 600 /usr/lib/ssl/certs/davmail.pem

```


Now open your DavMail configuration again:


```
sudo nano /etc/davmail.properties

```


Add the following configuration options to inform DavMail of the location of the pkcs12 file you just generated and the passphrase you set:


```
davmail.ssl.keystoreType=PKCS12
davmail.ssl.keystoreFile=/usr/lib/ssl/certs/davmail.p12
davmail.ssl.keyPass=password
davmail.ssl.keystorePass=password

```


Both davmail.ssl.keyPass and davmail.ssl.keystorePass should should have the same value. Save the configuration file.


DavMail is now configured to use your SSL certificate.


# Start DavMail


The Debian package we downloaded eariler does not contain an init script, so we must create our own.


Create a new file with your favorite text editor:


```
sudo nano /etc/init.d/davmail

```


Copy and paste the following into the file:


```
#! /bin/sh
### BEGIN INIT INFO
# Provides:          davmail
# Required-Start:    $remote_fs $syslog
# Required-Stop:     $remote_fs $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: DavMail Exchange gatway
# Description:       A gateway between Microsoft Exchange and open protocols.
    ### END INIT INFO

    # Author: Jesse TeKrony <jesse ~at~ jtekrony ~dot~ com>

    PATH=/sbin:/usr/sbin:/bin:/usr/bin
    DESC="Davmail Exchange gateway"
    NAME=davmail
    CONFIG=/etc/davmail.properties
    DAEMON=/usr/bin/$NAME
    DAEMON_ARGS="$CONFIG"
    PIDFILE=/var/run/$NAME.pid
    SCRIPTNAME=/etc/init.d/$NAME
    LOGFILE=/var/log/davmail.log

    # Exit if the package is not installed
    [ -x "$DAEMON" ] || exit 0

    # Read configuration variable file if it is present
    [ -r /etc/default/$NAME ] && . /etc/default/$NAME

    # Load the VERBOSE setting and other rcS variables
    . /lib/init/vars.sh

    # Define LSB log_* functions
    . /lib/lsb/init-functions

    #
    # Function that starts the daemon/service
    #
    do_start()
    {
        start-stop-daemon --start --quiet --pidfile $PIDFILE --exec $DAEMON --test > /dev/null \
            || return 1
        start-stop-daemon --start --quiet --pidfile $PIDFILE --exec $DAEMON -- \
            $DAEMON_ARGS >> $LOGFILE 2>&1 &
        [ $? != 0 ] && return 2
        echo $! > $PIDFILE
        exit 0
    }

    #
    # Function that stops the daemon/service
    #
    do_stop()
    {
        start-stop-daemon --stop --quiet --retry=TERM/30/KILL/5 --pidfile $PIDFILE
        RETVAL="$?"
        [ "$RETVAL" = 2 ] && return 2.
        start-stop-daemon --stop --quiet --oknodo --retry=0/30/KILL/5 --exec $DAEMON
        [ "$?" = 2 ] && return 2
        rm -f $PIDFILE
        return "$RETVAL"
    }

    case "$1" in
      start)
        [ "$VERBOSE" != no ] && log_daemon_msg "Starting $DESC" "$NAME"
        do_start
        case "$?" in
            0|1) [ "$VERBOSE" != no ] && log_end_msg 0 ;;
            2) [ "$VERBOSE" != no ] && log_end_msg 1 ;;
        esac
        ;;
      stop)
        [ "$VERBOSE" != no ] && log_daemon_msg "Stopping $DESC" "$NAME"
        do_stop
        case "$?" in
            0|1) [ "$VERBOSE" != no ] && log_end_msg 0 ;;
            2) [ "$VERBOSE" != no ] && log_end_msg 1 ;;
        esac
        ;;
      status)
           status_of_proc "$DAEMON" "$NAME" && exit 0 || exit $?
           ;;
      restart|force-reload)
        log_daemon_msg "Restarting $DESC" "$NAME"
        do_stop
        case "$?" in
          0|1)
            do_start
            case "$?" in
                0) log_end_msg 0 ;;
                1) log_end_msg 1 ;; # Old process is still running
                *) log_end_msg 1 ;; # Failed to start
            esac
            ;;
          *)
            # Failed to stop
            log_end_msg 1
            ;;
        esac
        ;;
      *)
        echo "Usage: $SCRIPTNAME {start|stop|status|restart| force-reload}" >&2
        exit 3
        ;;
    esac

```


Save and close the file.


Mark the script executable, start the service, and enable it at boot:


```
sudo chmod +x /etc/init.d/davmail
sudo service davmail start
sudo update-rc.d davmail defaults

```


# Client Configuration


Now that the server is running, you are ready to configure your E-mail clients. Create a new account, using the “manual” options of your E-mail client. Both the IMAP and SMTP server will be the domain name or IP address of your droplet, depending on what you used for the Common Name on your SSL certificate. The username for IMAP and SMTP will both be your E-mail address without the domain name. Example: Your E-mail is bob@yourcompany.com, so your username is bob. Make sure both IMAP and SMTP are set to use SSL/TLS and not STARTTLS.


You will get warnings from your E-mail clients because you are using a self-signed certificate. It is safe to accept the certificate in this case, because you are the one who created it.


Specific instructions for Thunderbird, Mac OSX, and iOS is available at DavMail’s website.


You should now be able to send/recieve E-mail using your Microsoft Exchange E-mail account using open technologies!


