# How To Stream Videos With Nginx and JWPlayer on CentOS 6

```Nginx``` ```CentOS```










# Status: Deprecated


This article covers a version of CentOS that is no longer supported. If you are currently operating a server running CentOS 6, we highly recommend upgrading or migrating to a supported version of CentOS.


Reason:
CentOS 6 reached end of life (EOL) on November 30th, 2020 and no longer receives security patches or updates. For this reason, this guide is no longer maintained.


See Instead:
This guide might still be useful as a reference, but may not work on other CentOS releases. If available, we strongly recommend using a guide written for the version of CentOS you are using.


# Step 1 - Install Nginx


```
rpm -ivh http://nginx.org/packages/centos/6/noarch/RPMS/nginx-release-centos-6-0.el6.ngx.noarch.rpm
yum -y install nginx
chkconfig nginx on && service nginx restart

```


# Step 2 - Download JWPlayer


You can get the latest version from their website


```
cd /usr/share/nginx/html
wget http://www.longtailvideo.com/download/jwplayer-3359.zip
unzip jwplayer-3359.zip

```


Now that you have your JWPlayer installed in /usr/share/nginx/html/jwplayer you would need to add an HTML file that calls it.


# Step 3 - Create an HTML file for playback


You can either embed this code into your existing webpages, or create a new file player.html and save it to /usr/share/nginx/html/player.html


From our previous article we have "big_buck_bunny_720p_surround.flv" flash file available for playback.


```
<html>
<body>
<script type="text/javascript" src="/jwplayer/jwplayer.js"></script>
<div id="myElement">Loading the player...</div>

<script type="text/javascript">
jwplayer("myElement").setup({ file: "big_buck_bunny_720p_surround.flv",
"autoStart": true });
</script>

</body>
</html>

```


Navigate over to your cloud server's IP and playback filename (http://198.199.88.112/player.html in our example):


![](https://assets.digitalocean.com/articles/community/JWPlayer.png)
And you are all done!


