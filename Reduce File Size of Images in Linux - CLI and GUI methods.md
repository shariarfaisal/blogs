# Reduce File Size of Images in Linux - CLI and GUI methods

```UNIX/Linux```

In this article, we talk about the different ways to reduce the file size of images in Linux. With the increase in the focus on the quality of images, the image file sizes have been increasing tremendously. There is a constant need of reducing the file size of such large images, therefore, we bring you an article which deals with the said task.


Let us quickly delve into the processes of reducing image file size.


# 1. Using the convert Command to Reduce File Size of Images in Linux


Before we move onto the application of this command, let us make sure it is present in the system.


The convert command comes under the ImageMagick package. Debian/Ubuntu users can install ImageMagick by running:


```
sudo apt install imagemagick

```


Installing ImageMagick package
Once the package is installed we can run man convert to take a look at the variety of operations supports by the command.


## Reducing by the quality of the image


The simplest way of reducing the size of the image is by degrading the quality of the image.


```
convert <INPUT_FILE> -quality 10% <OUTPUT_FILE>

```


Reducing image by quality
There is a significant reduce in the quality of the image using the convert command. In case we want to examine the new file size, we can do so by:


```
du -h jd_logo*

```


Check new image size
The du command provides the amount of disk used by files in Linux. In the above command, we display the amount of space occupied by all the versions of “jd_logo”.



## Reduce File Size of Images in Linux by pixels


The file size of image can be reduced if we reduce the amount of pixels it holds. For this purpose, we need to provide the new width and height.


```
convert <INPUT_FILE> -resize 200x200 <OUTPUT_FILE>

```


Reducing file size by pixels
The reduction in the quality of the reduced image can be observed when we stretch its dimensions.


The aspect ratio of the image is restored even though the dimensions provided in the command violated the original aspect ratio. The idea behind the conversion is that the reduced image must fit inside the specified dimensions.


In order to reduce the image into the exact dimensions, and neglecting the aspect ratio, '!' must be used after the resize parameter.


```
convert <INPUT_FILE> -resize 200x200! <OUTPUT_FILE>

```



## Converting the image format


Some websites only support specific file extensions, therefore convert command provides the facility to convert the image format.


```
convert <INPUT_FILE> <OUTPUT_FILE>

```


Convert ‘.png’ to ‘.jpg’
The reduction in quality is 92% if no parameter is provided. In the above snippet, we converted a ‘.png’ image file into a ‘.jpg’ file.


The convert command has hundreds of applications like rotating an image, applying effects or drawing stuff on an image. We can refer the manual pages by man convert to master the image formatting tool.


In order to convert multiple files, we need a bash script that runs a loop for all images. There is an alternative for processing multiple image files, which is mogrify that comes within the ImageMagick package.



# 2. Using the mogrify command


```
mogrify [OPTIONS] [FILE_LIST]

```


The main difference between convert and mogrify command is that mogrify command applies the operations on the original image file, whereas convert does not.


Moreover, mogrify command supports expressions to queue in multiple files. For instance:


```
mogrify -quality 10 *.jpg

```


Reducing file size of images
The applications for convert and mogrify are identical as they are derived from the same package.



# 3. Using Pngcrush for PNG files


pngcrush is a PNG (Portable Network Graphics) file optimizer. It reduces the file size of the image by passing it through various compression methods and filters.


Debian/Ubuntu users can run the following command for installation.


```
sudo apt get install pngcrush

```


Users of other Linux distributions can install it using their standard installation commands followed by pngcrush.


After the installation is done, we can reduce the size of PNG file by running:


```
pngcrush -brute <INPUT_FILE> <OUTPUT_FILE>

```


Reducing PNG file size
The '-brute' option takes the file through 114 filter/compression methods. The extended process consumes few seconds. Instead of applying the brute force approach, users can select filters, levels and strategies for optimization.


The types of filters and other properties can be learnt through the manual pages - man pngcrush.



# 4. Using Jpegoptim for JPG files


jpegoptim is a JPG (Joint Photographic Group) file compressor. This command supports percentage and target file size as a parameter to reduce the image size.


The installation is pretty simple.


```
sudo apt install jpegoptim

```


Once the installation is finished, we can run:


```
jpegoptim --size=<TARGET_SIZE> <INPUT_FILE>

```


Reducing JPG file size
The jpegoptim utility overwrites the original image therefore it is advised to keep a backup image file. The best feature of this tool is that it accepts the target file size, which can be a life-saver for uploading images of specific sizes.


In the above figure, we compressed a 260 KB file into a 20KB image.


Difference in Images
The quality of the image is intact, even though there is a massive 90% reduction in size. The command supports compression on the basis of percentages too.


We can learn more about the command from the manual pages through - man jpegoptim.



# 5. Using the Trimage GUI Tool


The trimage GUI Tool is a basic drag and drop software. The added files are automatically compressed to the possible lossless file size.


The installation is similar to the previous methods.


```
sudo apt install trimage

```


After the installation is complete, we can access it by searching “trimage” on the system. The trimage window looks like the following image:


Reducing image size using ‘trimage’
The supported columns are:


- Name of the file
- Size of the original image
- Size of the converted image
- Percentage of compression

The tool overwrites the original image. The compression is minimal due to the fact that the compression is lossless.


GIMP (GNU Image Manipulation Program) is a good alternative for a GUI-based image size reduction, but it is definitely an over-kill.



# Conclusion


The simplest and the most effective way to reduce file size of images in Linux is using the commands provided by the ImageMagick package.


We hope the article was interesting as well as informative. Thank you for reading.



# References


Pngcrush Official Website


Trimage Official Website


