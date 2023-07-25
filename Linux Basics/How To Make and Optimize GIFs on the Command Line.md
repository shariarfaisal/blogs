# How To Make and Optimize GIFs on the Command Line

```Linux Basics```

## Introduction


Along with JPG and PNG, GIFs are one of the most common image formats that have been circulating since the 1990s. Unlike JPG and PNG, GIFs can contain multiple frames of animation, and the humble “animated GIF” is a ubiquitous building block of the internet.


GIFs are actually an old technology, and they are now less efficient than embedding web videos in many contexts. This is because most web video uses modern video compression technologies and more popular modern codecs than GIF. Codecs are used to encode and decode videos, and most platforms have dedicated hardware to play those codecs. GIFs, on the other hand, are always decoded directly with the CPU. The CPU overhead of a low resolution GIF with only a few frames of animation is negligible, but you could technically create a GIF with a comparable resolution and framerate to a YouTube video, and you would be surprised by how many of your system resources it consumes.


However, GIFs are still useful because they are considered images and not videos. Because of the way the web and other applications work, that means they will render and animate automatically in many more contexts, and do not need to be embedded or linked separately. This can be handy for everything from reaction images to interactive fiction development or other presentation formats.


In this tutorial, you will try out several tools for creating GIFs from video clips, optimizing them for size and quality, and ensuring you can use them in many contexts. You can also combine these tools to integrate into another application stack.


## Prerequisites


This tutorial will provide installation instructions for a Ubuntu 22.04 server. You can set this up by following our guide on Initial Server Setup with Ubuntu 22.04.


You will also need to have installed the Homebrew package manager to install one of the tools in this tutorial.


# Step 1 — Installing ffmpeg, Gifski, and Gifsicle


In this tutorial, you will need three tools to follow along with the examples. The first is: ffmpeg for cutting and manipulating videos, then Gifski for creating GIFs, and finally Gifsicle for optimizing and further manipulating your GIFs. These tools are available on most platforms.


Both ffmpeg and gifsicle are available in Ubuntu’s default repositories, and can be installed with the apt package manager. Begin by updating your package sources with apt update:


```
sudo apt update


```


Then, install the ffmpeg and gifsicle packages with apt install:


```
sudo apt install ffmpeg gifsicle


```


The last tool, gifski, is available via Homebrew. Install it with brew install (this will take a few minutes as Homebrew installs other dependencies):


```
brew install gifski


```


You now have all the necessary tools installed on your machine. Next, you’ll start by acquiring a sample video to create a GIF from.


# Step 2 — Downloading and Examining a Sample Video


You can make a GIF from any existing video clip. If you don’t already have one that you want to use, you can use our video on Introducing App Platform by DigitalOcean as a starting point.


You can download a copy of this video from elsewhere on our servers using curl:


```
curl -O https://deved-images.nyc3.digitaloceanspaces.com/gif-cli/app-platform.webm


```


curl is a command line tool for making all kinds of web requests. Using the -O flag with a URL directs curl to download a remote file and store it with the same filename locally.


Now that you have a copy of the video locally, you can check some of its metadata. This will be relevant when trying to create a high-quality GIF. When you installed ffmpeg earlier, it also came with a command called ffprobe, which can be used to check resolution, framerate, and other information in media files. Review these details by running ffprobe on the app-platform.webm video you downloaded:


```
ffprobe app-platform.webm


```


```
Output…
Input #0, matroska,webm, from 'app-platform.webm':
  Metadata:
    ENCODER         : Lavf59.27.100
  Duration: 00:01:59.04, start: -0.007000, bitrate: 1362 kb/s
  Stream #0:0(eng): Video: vp9 (Profile 0), yuv420p(tv, bt709), 1920x1080, SAR 1:1 DAR 16:9, 25 fps, 25 tbr, 1k tbn (default)
    Metadata:
      DURATION        : 00:01:59.000000000
  Stream #0:1(eng): Audio: opus, 48000 Hz, stereo, fltp (default)
    Metadata:
      DURATION        : 00:01:59.041000000

```


The output lists any streams contained in the file (usually one video and at least one audio stream), as well as the sample rate, codecs, and other properties of the streams. From the highlighted information in the output, you learn that this video is encoded to 1080p resolution, and is played at 25 frames per second. It is also almost two minutes long, which you may have learned from watching it on YouTube, and is probably too long for one GIF!


This is enough information to move on to the next step where you’ll cut a clip out of this video to make a GIF from it.


# Step 3 — Cutting a Clip from a Video


You now have a two minute long video file that you know the properties of. The only thing you need to do before cutting it to a GIF is extract a shorter clip from it.


It isn’t very convenient to play a video in a terminal shell, so you can watch along with the video on YouTube to find an ideal place to cut. In this tutorial, you’ll cut from 00:00:09 to 00:00:12, which produces a pretty smooth animation:





You can make that cut by passing the app-platform.webm video to ffmpeg:


```
ffmpeg -ss 00:00:09 -to 00:00:12 -i app-platform.webm -c copy clip.webm


```


This command is broken down by:


- -ss 00:00:09 -to 00:00:12 is how ffmpeg understands timecodes. In this case, cutting from a starting position to an ending position in the clip. You can also clip based on duration, or to fractions of a second.
- -i app-platform.webm is the path to your input file, preceded by -i.
- -c copy is where you would normally specify an output audio or video codec to ffmpeg. Using copy in lieu of a codec skips reencoding the video at all, which can be much quicker and avoid any loss in quality, as long as you don’t need to target a different output format. Because you’re making this clip into a GIF later anyway, preserving the native input format is fine, and saves time.
- clip.webm is the path to your output file.

This creates a new threesecond video called clip.webm. You can verify that it exists and check its size using ls -lh:


```
ls -lh clip.webm


```


```
Output-rw-r--r-- 1 sammy sammy 600K Nov 16 14:27 clip.webm

```


It turns out that three seconds of video are only 600K large. This will make a good point of comparison when creating your GIF in the next step.



Note: If you are working on a local machine, you can use an open-source GUI tool called Lossless Cut to perform this same operation. Lossless Cut is particularly useful because it runs the same ffmpeg commands to quickly extract clips from a video based on timecodes, without needing to re-encode the video. Unlike running ffmpeg on its own on the command line, Lossless Cut includes a built-in video player and navigation interface.

# Step 4 — Converting a Video to a GIF


Now that you have a three-second long video and an upper limit in mind for its frame rate and resolution, you can make a GIF from it. If you were developing an automated conversion pipeline for uploading videos to GIFs, it could be helpful to automatically extract the video resolution and framerate from ffprobe, to pass directly to these next few commands. In this tutorial, you’ll onlybe hardcoding some sensible resolution and framerate values for your output.


You have a few options for making a GIF on the command line. You can do it with ffmpeg by itself, but the syntax can be very hard to change or understand:


```
ffmpeg -filter_complex "[0:v] fps=12,scale=w=540:h=-1,split [a][b];[a] palettegen [p];[b][p] paletteuse" -i clip.webm ffmpeg-sample.gif


```


Note that in this example, you’ve cut the resolution and the framerate of the clip down by approximately half, to 12fps and 540p resolution. This is usually a good starting point for a GIF. Because GIFs are treated like images, they are downloaded in full when a web page is loaded, and unlike videos, they have no concept of gradually streaming in at a lower resolution. Using CDNs can help optimize the delivery of any static site assets like these, but you should still try to avoid serving huge images without a particular reason. Therefore, you should always avoid making your GIFs larger than necessary; under 3M is usually good practice for images. You can check the file size of your new GIF using ls -lh:


```
ls -lh ffmpeg-sample.gif


```


```
Output-rw-r--r-- 1 sammy sammy 2.0M Nov 16 14:28 ffmpeg-sample.gif

```


You’ve created a 2M GIF this way. While this is good, you can produce an even better result using less complicated syntax with gifski. Try this gifski command:


```
gifski --fps 12 --width 540 -o gifski-sample.gif clip.webm


```


Notice how you only need to preserve the important details — framerate and resolution — along with your input and output file names. Check the output file afterward:


```
ls -lh gifski-sample.gif


```


```
Output-rw-r--r-- 1 sammy sammy 1.3M Nov 16 14:33 gifski-sample.gif

```


This one is only 1.3M, a significant improvement at the same quality level. At this point, you might be tempted to try making a full-framerate, full-resolution version of the GIF. Here’s a point of comparison:


```
gifski --fps 25 --width 1080 -o gifski-high.gif clip.webm


```


Check the size of this last test file:


```
ls -lh gifski-high.gif


```


```
Output-rw-r--r-- 1 sammy sammy 6.9M Nov 16 14:37 gifski-high.gif

```


6.9M is definitely too large, considering your original video clip was only 0.6M! Remember, GIFs are not particularly efficient compared to modern video codecs. You need to make some minor sacrifices when encoding them down to a reasonable file size. In the final step of this tutorial, you’ll further optimize your GIFs.



Note: At any point in this tutorial, if you are working on a remote server, you can download and inspect your GIFs locally, or even move them into a web-accessible directory so you can view them in a web browser. This will allow you to get a visual reference for the animation quality.

# Step 5 — Optimizing, Validating, and Viewing a GIF


For the last step of this tutorial, you’ll use gifsicle to refine your GIFs. gifsicle is to GIFs what ffmpeg is to audio and video: it can do almost anything, but can be quite complicated as a result. For that reason, you can stick to gifski for actually creating GIFs, and focus on a few gifsicle commands to improve or manipulate them.


Start by running a standard gifsicle optimization command:


```
gifsicle -O3 --lossy=80 --colors 256 gifski-sample.gif -o optimized.gif


```


In this command, you provided the -O3 option for the most aggressive optimization, --lossy 80 to allow an up to 20% loss in image quality from the source input, and --colors 256 to use a maximum of 256 colors in your output image. This will produce a higher quality image than you may expect, with almost no visible loss in image quality, because GIFs do not use modern inter-frame video compression the way that video codecs do, nor do they use JPEG-style image compression techniques by default. Also, in this context, 256 colors refers to any 256 color-palette based on what’s already in your GIF, rather than a restricted palette of only the most common 256 colors, as you may otherwise associate with small color palettes. In general, GIF compression is not very perceptible.


As with the last step, check the size of optimized.gif:


```
ls -lh optimized.gif


```


```
Output-rw-r--r-- 1 sammy sammy 935K Nov 16 14:44 optimized.gif

```


This last step has successfully reduced the file size to only slightly larger than the original video, a very reasonable 935K for an animated image. This is the same optimized GIF that was displayed earlier in this tutorial.


You can refer to the Gifsicle manual to learn about  other ways of manipulating your GIFs. For example, you can “explode” the GIF into multiple image files, one for each frame of animation:


```
gifsicle --explode optimized.gif


```


This creates multiple files named optimized.gif.000, optimized.gif.001, and so on, for every individual image:


```
ls -lh optimized*


```


```
Output-rw-r--r-- 1 sammy sammy 935K Nov 16 14:46 optimized.gif
-rw-r--r-- 1 sammy sammy  20K Nov 16 14:54 optimized.gif.000
-rw-r--r-- 1 sammy sammy  17K Nov 16 14:54 optimized.gif.001
-rw-r--r-- 1 sammy sammy  22K Nov 16 14:54 optimized.gif.002
-rw-r--r-- 1 sammy sammy  22K Nov 16 14:54 optimized.gif.003
…

```


You can also rotate your GIF using the --rotate-90 or --rotate-180 options:


```
gifsicle --rotate-90 optimized.gif -o rotated.gif


```


Despite their inefficiency, GIFs remain useful because they can be used nearly anywhere. When you need a short clip to animate automatically, or you specifically need an image and not a video format, sometimes there’s no substitute for a good old GIF.


# Conclusion


In this tutorial, you used multiple tools to create a well-optimized GIF from an existing video. You also reviewed the ecosystem of open-source video manipulation and GIF manipulation tools, as well as some other options for further editing your GIFs. GIFs are an interesting, anachronistic technology.n many ways, they are not modern, but they still have no real replacement in some contexts, and the tools for working with GIFs are robust and well-maintained. With that being said, go forth and use your GIFs wisely.


Next, you may want to learn how to build a media processing API in Node.js.


