# How To Convert Videos with FFMpeg On CentOS 6

```CentOS```










# Status: Deprecated


This article covers a version of CentOS that is no longer supported. If you are currently operating a server running CentOS 6, we highly recommend upgrading or migrating to a supported version of CentOS.


Reason:
CentOS 6 reached end of life (EOL) on November 30th, 2020 and no longer receives security patches or updates. For this reason, this guide is no longer maintained.


See Instead:
This guide might still be useful as a reference, but may not work on other CentOS releases. If available, we strongly recommend using a guide written for the version of CentOS you are using.


FFMpeg is a popular program for converting and manipulating audio/video files.


We will need to spin up a CentOS 6.4 x64 cloud server:


![](https://assets.digitalocean.com/articles/community/FFMpeg-Droplet.png)
# Step 1 - Install ATRPMS Repository


```
rpm --import http://packages.atrpms.net/RPM-GPG-KEY.atrpms
rpm -ivh http://dl.atrpms.net/el6-x86_64/atrpms/stable/atrpms-repo-6-7.el6.x86_64.rpm

```


# Step 2 - Install FFMpeg from ATRPMS Repository


```
yum -y --enablerepo=atrpms install ffmpeg

```


Verify that you have FFMpeg installed:


```
ffmpeg -version

```


![](https://assets.digitalocean.com/articles/community/FFMpeg.png)
To get a list of supported formats:


```
ffmpeg -formats

```


# Step 3 - Convert your Videos


Once you have uploaded your video you can begin converting it to various formats.


For our example, we will download "Big Buck Bunny 720p MP4" video and convert it.


```
wget "http://mirrorblender.top-ix.org/peach/bigbuckbunny_movies/big_buck_bunny_720p_surround.avi"

```


In 720p MP4 format this video is 317MB:


```
[root@FFMpeg ~]# ls -lah big_buck_bunny_720p_surround.avi 
-rw-r--r-- 1 root root 317M May  6  2008 big_buck_bunny_720p_surround.avi

```


## Convert from MP4 to H264


```
ffmpeg -i big_buck_bunny_720p_surround.avi -vcodec libx264 big_buck_bunny_720p_surround-H264.avi

```


Once converted from MP4 to H264 this video is 118MB:


```
[root@FFMpeg ~]# ls -lah big_buck_bunny_720p_surround-H264.avi 
-rw-r--r-- 1 root root 118M May 30 23:40 big_buck_bunny_720p_surround-H264.avi

```


## Convert from H264 to FLV


```
ffmpeg -i libx264 big_buck_bunny_720p_surround-H264.avi -vcodec libx264 -ar 44100 -f flv libx264 big_buck_bunny_720p_surround.flv

```


The FLV version is 102MB:


```
[root@FFMpeg ~]# ls -lah big_buck_bunny_720p_surround.flv 
-rw-r--r-- 1 root root 102M May 31 00:06 big_buck_bunny_720p_surround.flv

```


You can stream these files with JWPlayer as described in our next article.


