# How To Process Images in Node js with Sharp

```Node.js``` ```Custom Images``` ```Development```

The author selected the Diversity in Tech Fund to receive a donation as part of the Write for DOnations program.


## Introduction


Digital image processing is a method of using a computer to analyze and manipulate images. The process involves reading an image, applying methods to alter or enhance the image, and then saving the processed image. It’s common for applications that handle user-uploaded content to process images. For example, if you’re writing a web application that allows users to upload images, users may upload unnecessary large images. This can negatively impact the application load speed, and also waste your server space. With image processing, your application can resize and compress all the user-uploaded images, which can significantly improve your application performance and save your server disk space.


Node.js has an ecosystem of libraries you can use to process images, such as sharp, jimp, and gm module. This article will focus on the sharp module. sharp is a popular Node.js image processing library that supports various image file formats, such as JPEG, PNG, GIF, WebP, AVIF, SVG and TIFF.


In this tutorial, you’ll use sharp to read an image and extract its metadata, resize, change an image format, and compress an image. You will then crop, grayscale, rotate, and blur an image. Finally, you will composite images, and add text on an image. By the end of this tutorial, you’ll have a good understanding of how to process images in Node.js.


# Prerequisites


To complete this tutorial, you’ll need:


- 
Node.js set up in your local development environment. You can follow How to Install Node.js and Create a Local Development Environment to learn how to install Node.js and npm on your system.

- 
Basic knowledge of how to write, and run a Node.js program. You can follow How To Write and Run Your First Program in Node.js to learn the basics.

- 
Basic understanding of asynchronous programming in JavaScript. Follow Understanding the Event Loop, Callbacks, Promises, and Async/Await in JavaScript to review asynchronous programming.


# Step 1 — Setting Up the Project Directory and Downloading Images


Before you start writing your code, you need to create the directory that will contain the code and the images you’ll use in this article.


Open your terminal and create the directory for the project using the mkdir command:


```
mkdir process_images


```


Move into the newly created directory using the cd command:


```
cd process_images


```


Create a package.json file using npm init command to keep track of the project dependencies:


```
npm init -y


```


The -y option tells npm to create the default package.json file.


Next, install sharp as a dependency:


```
npm install sharp


```


You will use the following three images in this tutorial:







Next, download the images in your project directory using the curl command.


Use the following command to download the first image. This will download the image as sammy.png:


```
curl -O https://assets.digitalocean.com/how-to-process-images-in-node-js-with-sharp/sammy.png


```


Next, download the second image with the following command. This will download the image as underwater.png:


```
curl -O https://assets.digitalocean.com/how-to-process-images-in-node-js-with-sharp/underwater.png


```


Finally, download the third image using the following command. This will download the image as sammy-transparent.png:


```
curl -O https://assets.digitalocean.com/how-to-process-images-in-node-js-with-sharp/sammy-transparent.png


```


With the project directory and the dependencies set up, you’re now ready to start processing images.


# Step 2 — Reading Images and Outputting Metadata


In this section, you’ll write code to read an image and extract its metadata. Image metadata is text embedded into an image, which includes information about the image such as its type, width, and height.


To extract the metadata, you’ll first import the sharp module, create an instance of sharp, and pass the image path as an argument. After that, you’ll chain the metadata() method to the instance to extract the metadata and log it into the console.


To do this, create and open readImage.js file in your preferred text editor. This tutorial uses a terminal text editor called nano:


```
nano readImage.js


```


Next, require in sharp at the top of the file:


process_images/readImage.js
```
const sharp = require("sharp");

```


sharp is a promise-based image processing module. When you create a sharp instance, it returns a promise. You can resolve the promise using the then method or use async/await, which has a cleaner syntax.


To use async/await syntax, you’ll need to create an asynchronous function by placing the async keyword at the beginning of the function. This will allow you to use the await keyword inside the function to resolve the promise returned when you read an image.


In your readImage.js file, define an asynchronous function, getMetadata(), to read the image, extract its metadata, and log it into the console:


process_images/readImage.js
```
const sharp = require("sharp");

async function getMetadata() {
  const metadata = await sharp("sammy.png").metadata();
  console.log(metadata);
}


```


getMetadata() is an asynchronous function given the async keyword you defined before the function label. This lets you use the await syntax within the function. The getMetadata() function will read an image and return an object with its metadata.


Within the function body, you read the image by calling sharp() which takes the image path as an argument, here with sammy.png.


Apart from taking an image path, sharp() can also read image data stored in a Buffer, Uint8Array, or Uint8ClampedArray provided the image is JPEG, PNG, GIF, WebP, AVIF, SVG or TIFF.


Now, when you use sharp() to read the image, it creates a sharp instance. You then chain the metadata() method of the sharp module to the instance. The method returns an object containing the image metadata, which you store in the metadata variable and log its contents using console.log().


Your program can now read an image and return its metadata. However, if the program throws an error during execution, it will crash. To get around this, you need to capture the errors when they occur.


To do that, wrap the code within the getMetadata() function inside a try...catch block:


process_images/readImage.js
```
const sharp = require("sharp");

async function getMetadata() {
  try {
    const metadata = await sharp("sammy.png").metadata();
    console.log(metadata);
  } catch (error) {
    console.log(`An error occurred during processing: ${error}`);
  }
}

```


Inside the try block, you read an image, extract and log its metadata. When an error occurs during this process, execution skips to the catch section and logs the error preventing the program from crashing.


Finally, call the getMetadata() function by adding the highlighted line:


process_images/readImage.js
```

const sharp = require("sharp");

async function getMetadata() {
  try {
    const metadata = await sharp("sammy.png").metadata();
    console.log(metadata);
  } catch (error) {
    console.log(`An error occurred during processing: ${error}`);
  }
}

getMetadata();

```


Now, save and exit the file. Enter y to save the changes you made in the file, and confirm the file name by pressing ENTER or RETURN key.


Run the file using the node command:


```
node readImage.js


```


You should see an output similar to this:


```
Output{
  format: 'png',
  width: 750,
  height: 483,
  space: 'srgb',
  channels: 3,
  depth: 'uchar',
  density: 72,
  isProgressive: false,
  hasProfile: false,
  hasAlpha: false
}

```


Now that you’ve read an image and extracted its metadata, you’ll now resize an image, change its format, and compress it.


# Step 3 — Resizing, Changing Image Format, and Compressing Images


Resizing is the process of altering an image dimension without cutting anything from it, which affects the image file size. In this section, you’ll resize an image, change its image type, and compress the image. Image compression is the process of reducing an image file size without losing quality.


First, you’ll chain the resize() method from the sharp instance to resize the image, and save it in the project directory. Second, you’ll chain the format() method to the resized image to change its format from png to jpeg. Additionally, you will pass an option to the format() method to compress the image and save it to the directory.


Create and open resizeImage.js file in your text editor:


```
nano resizeImage.js


```


Add the following code to resize the image to 150px width and 97px height:


process_images/resizeImage.js
```
const sharp = require("sharp");

async function resizeImage() {
  try {
    await sharp("sammy.png")
      .resize({
        width: 150,
        height: 97
      })
      .toFile("sammy-resized.png");
  } catch (error) {
    console.log(error);
  }
}

resizeImage();

```


The resizeImage() function chains the sharp module’s resize() method to the sharp instance. The method takes an object as an argument. In the object, you set the image dimensions you want using the width and height property. Setting the width to 150 and the height to 97 will make the image 150px wide, and 97px tall.


After resizing the image, you chain the sharp module’s toFile() method, which takes the image path as an argument. Passing sammy-resized.png as an argument will save the image file with that name in the working directory of your program.


Now, save and exit the file. Run your program in the terminal:


```
node resizeImage.js


```


You will get no output, but you should see a new image file created with the name sammy-resized.png in the project directory.


Open the image on your local machine. You should see an image of Sammy 150px wide and 97px tall:





Now that you can resize an image, next you’ll convert the resized image format from png to jpeg, compress the image, and save it in the working directory. To do that, you will use toFormat() method, which you’ll chain after the resize() method.


Add the highlighted code to change the image format to jpeg and compress it:


process_images/resizeImage.js
```
const sharp = require("sharp");

async function resizeImage() {
  try {
    await sharp("sammy.png")
      .resize({
        width: 150,
        height: 97
      })
      .toFormat("jpeg", { mozjpeg: true })
      .toFile("sammy-resized-compressed.jpeg");
  } catch (error) {
    console.log(error);
  }
}

resizeImage();

```


Within the resizeImage() function, you use the toFormat() method of the sharp module to change the image format and compress it. The first argument of the toFormat() method is a string containing the image format you want to convert your image to. The second argument is an optional object containing output options that enhance and compress the image.


To compress the image, you pass it a mozjpeg property that holds a boolean value. When you set it to true, sharp uses mozjpeg defaults to compress the image without sacrificing quality. The object can also take more options; see the sharp documentation for more details.



Note: Regarding the toFormat() method’s second argument, each image format takes an object with different properties. For example, mozjpeg property is accepted only on JPEG images.
However, other image formats have equivalents options such quality, compression, and lossless. Make sure to refer to the documentation to know what kind of options are acceptable for the image format you are compressing.

Next, you pass the toFile() method a different filename to save the compressed image as sammy-resized-compressed.jpeg.


Now, save and exit the file, then run your code with the following command:


```
node resizeImage.js


```


You will receive no output, but an image file sammy-resized-compressed.jpeg is saved in your project directory.


Open the image on your local machine and you will see the following image:





With your image now compressed, check the file size to confirm your compression is successful. In your terminal, run the du command to check the file size for sammy.png:


```
du -h sammy.png


```


-h option produces human-readable output showing you the file size in kilobytes, megabytes and many more.


After running the command, you should see an output similar to this:


```
Output120K    sammy.png

```


The output shows that the original image is 120 kilobytes.


Next, check the file size for sammy-resized.png:


```
du -h sammy-resized.png


```


After running the command, you will see the following output:


```
Output8.0K     sammy-resized.png

```


sammy-resized.png is 8 kilobytes down from 120 kilobytes. This shows that the resizing operation affects the file size.


Now, check the file size for sammy-resized-compressed.jpeg:


```
du -h sammy-resized-compressed.jpeg


```


After running the command, you will see the following output:


```
Output4.0K    sammy-resized-compressed.jpeg

```


The sammy-resized-compressed.jpeg is now 4 kilobytes down from 8 kilobytes, saving you 4 kilobytes, showing that the compression worked.


Now that you’ve resized an image, changed its format and compressed it, you will crop and grayscale the image.


# Step 4 — Cropping and Converting Images to Grayscale


In this step, you will crop an image, and convert it to grayscale. Cropping is the process of removing unwanted areas from an image. You’ll use the extend() method to crop the sammy.png image. After that, you’ll chain the grayscale() method to the cropped image instance and convert it to grayscale.


Create and open cropImage.js in your text editor:


```
nano cropImage.js


```


In your cropImage.js file, add the following code to crop the image:


process_images/cropImage.js
```
const sharp = require("sharp");

async function cropImage() {
  try {
    await sharp("sammy.png")
      .extract({ width: 500, height: 330, left: 120, top: 70  })
      .toFile("sammy-cropped.png");
  } catch (error) {
    console.log(error);
  }
}

cropImage();

```


The cropImage() function is an asynchronous function that reads an image and returns your image cropped. Within the try block, a sharp instance will read the image. Then, the sharp module’s extract() method chained to the instance takes an object with the following properties:


- width: the width of the area you want to crop.
- height: the height of the area you want to crop.
- top: the vertical position of the area you want to crop.
- left: the horizontal position of the area you want to crop.

When you set the width to 500 and the height to 330, imagine that sharp creates a transparent box on top of the image you want to crop. Any part of the image that fits in the box will remain, and the rest will be cut:





The top and left properties control the position of the box. When you set left to 120, the box is positioned 120px from the left edge of the image, and setting top to 70 positions the box 70px from the top edge of the image.


The area of the image that fits within the box will be extracted out and saved into sammy-cropped.png as a separate image.


Save and exit the file. Run the program in the terminal:


```
node cropImage.js


```


The output won’t be shown but the image sammy-cropped.png will be saved in your project directory.


Open the image on your local machine. You should see the image cropped:





Now that you cropped an image, you will convert the image to grayscale. To do that, you’ll chain the grayscale method to the sharp instance. Add the highlighted code to convert the image to grayscale:


process_images/cropImage.js
```
const sharp = require("sharp");

async function cropImage() {
  try {
    await sharp("sammy.png")
      .extract({ width: 500, height: 330, left: 120, top: 70 })
      .grayscale()
      .toFile("sammy-cropped-grayscale.png");
  } catch (error) {
    console.log(error);
  }
}

cropImage();

```


The cropImage() function converts the cropped image to grayscale by chaining the sharp module’s grayscale() method to the sharp instance. It then saves the image in the project directory as sammy-cropped-grayscale.png.


Press CTRL+X to save and exit the file.


Run your code in the terminal:


```
node cropImage.js


```


Open sammy-cropped-grayscale.png on your local machine. You should now see the image in grayscale:





Now that you’ve cropped and extracted the image, you’ll work with rotating and blurring it.


# Step 5 — Rotating and Blurring Images


In this step, you’ll rotate the sammy.png image at a 33 degrees angle. You’ll also apply a gaussian blur on the rotated image. A gaussian blur is a technique of blurring an image using the Gaussian function, which reduces the noise level and detail on an image.


Create a rotateImage.js file in your text editor:


```
nano rotateImage.js


```


In your rotateImage.js file, write the following code block to create a function that rotates sammy.png to an angle of 33 degrees:


process_images/rotateImage.js
```
const sharp = require("sharp");

async function rotateImage() {
  try {
    await sharp("sammy.png")
      .rotate(33, { background: { r: 0, g: 0, b: 0, alpha: 0 } })
      .toFile("sammy-rotated.png");
  } catch (error) {
    console.log(error);
  }
}

rotateImage();

```


The rotateImage() function is an asynchronous function that reads an image and will return the image rotated to an angle of 33 degrees. Within the function, the rotate() method of the sharp module takes two arguments. The first argument is the rotation angle of 33 degrees. By default, sharp makes the background of the rotated image black. To remove the black background, you pass an object as a second argument to make the background transparent.


The object has a background property which holds an object defining the RGBA color model. RGBA stands for red, green, blue, and alpha.


- 
r: controls the intensity of the red color. It accepts an integer value of 0 to 255. 0 means the color is not being used, and 255 is red at its highest.

- 
g: controls the intensity of the green color. It accepts an integer value of 0-255. 0 means that the color green is not used, and 255 is green at its highest.

- 
b: controls the intensity of blue. It also accepts an integer value between 0 and 255. 0 means that the blue color isn’t used, and 255 is blue at its highest.

- 
alpha: controls the opacity of the color defined by r, g, and b properties. 0 or 0.0 makes the color transparent and 1 or 1.1 makes the color opaque.


For the alpha property to work, you must make sure you define and set the values for r, g, and b. Setting the r, g, and b values to 0 creates a black color. To create a transparent background, you must define a color first, then you can set alpha to 0 to make it transparent.


Now, save and exit the file. Run your script in the terminal:


```
node rotateImage.js


```


Check for the existence of sammy-rotated.png in your project directory. Open it on your local machine.


You should see the image rotated to an angle of 33 degrees:





Next, you’ll blur the rotated image. You’ll achieve that by chaining the blur() method to the sharp instance.


Enter the highlighted code below to blur the image:


process_images/rotateImage.js
```
const sharp = require("sharp");

async function rotateImage() {
  try {
    await sharp("sammy.png")
      .rotate(33, { background: { r: 0, g: 0, b: 0, alpha: 0 } })
      .blur(4)
      .toFile("sammy-rotated-blurred.png");
  } catch (error) {
    console.log(error);
  }
}

rotateImage();

```


The rotateImage() function now reads the image, rotate it, and applies a gaussian blur to the image. It applies a gaussian blur to the image using the sharp module’s blur() method. The method accepts a single argument containing a sigma value between 0.3 and 1000. Passing it 4 will apply a gaussian blur with a sigma value of 4. After the image is blurred, you define a path to save the blurred image.


Your script will now blur the rotated image with a sigma value of 4. Save and exit the file, then run the script in your terminal:


```
node rotateImage.js


```


After running the script, open sammy-rotated-blurred.png file on your local machine. You should now see the rotated image blurred:





Now that you’ve rotated and blurred an image, you’ll composite an image over another.


# Step 6 — Compositing Images Using composite()


Image Composition is a process of combining two or more separate pictures to create a single image. This is done to create effects that borrow the best elements from the different photos. Another common use case is to watermark an image with a logo.


In this section, you’ll composite sammy-transparent.png over the underwater.png. This will create an illusion of sammy swimming deep in the ocean. To composite the images, you’ll chain the composite() method to the sharp instance.


Create and open the file compositeImage.js in your text editor:


```
nano compositeImages.js


```


Now, create a function to composite the two images by adding the following code in the compositeImages.js file:


process_images/compositeImages.js
```
const sharp = require("sharp");

async function compositeImages() {
  try {
    await sharp("underwater.png")
      .composite([
        {
          input: "sammy-transparent.png",
          top: 50,
          left: 50,
        },
      ])
      .toFile("sammy-underwater.png");
  } catch (error) {
    console.log(error);
  }
}

compositeImages()

```


The compositeImages() function reads the underwater.png image first. Next, you chain the composite() method of the sharp module, which takes an array as an argument. The array contains a single object that reads the sammy-transparent.png image. The object has the following properties:


- input: takes the path of the image you want to composite over the processed image. It also accepts a Buffer, Uint8Array, or Uint8ClampedArray as input.
- top: controls the vertical position of the image you want to composite over. Setting top to 50 offsets the sammy-transparent.png image 50px from the top edge of the underwater.png image.
- left: controls the horizontal position of the image you want to composite over another. Setting left to 50 offsets the sammy-transparent.png 50px from the left edge of the underwater.png image.

The composite() method requires an image of similar size or smaller to the processed image.


To visualize what the composite() method is doing, think of it like its creating a stack of images. The sammy-transparent.png image is placed on top of underwater.png image:





The top and left values positions the sammy-transparent.png image relative to the underwater.png image.


Save your script and exit the file. Run your script to create an image composition:


```
node compositeImages.js

```


Open sammy-underwater.png in your local machine. You should now see the sammy-transparent.png composited over the underwater.png image:





You’ve now composited images using the composite() method. In the next step, you’ll use the composite() method to add text to an image.


# Step 7 — Adding Text on an Image


In this step, you’ll write text on an image. At the time of writing, sharp doesn’t have a native way of adding text to an image. To add text, first, you’ll write code to draw text using Scalable Vector Graphics(SVG). Once you’ve created the SVG image, you’ll write code to composite the image with the sammy.png image using the composite method.


SVG is an XML-based markup language for creating vector graphics for the web. You can draw text, or shapes such as circles, triangles, and as well as draw complex shapes such as illustrations, logos, etc. The complex shapes are created with a graphic tool like Inkscape which generates the SVG code. The SVG shapes can be rendered and scaled to any size without losing quality.


Create and open the addTextOnImage.js file in your text editor.


```
nano addTextOnImage.js


```


In your addTextOnImage.js file, add the following code to create an SVG container:


process_images/addTextOnImage.js
```
const sharp = require("sharp");

async function addTextOnImage() {
  try {
    const width = 750;
    const height = 483;
    const text = "Sammy the Shark";

    const svgImage = `
    <svg width="${width}" height="${height}">
    </svg>
    `;
  } catch (error) {
    console.log(error);
  }
}

addTextOnImage();

```


The addTextOnImage() function defines four variables: width, height, text, and svgImage. width holds the integer 750, and height holds the integer 483. text holds the string Sammy the Shark. This is the text that you’ll draw using SVG.


The svgImage variable holds the svg element. The svg element has two attributes: width and height that interpolates the width and height variables you defined earlier. The svg element creates a transparent container according to the given width and height.


You gave the svg element a width of 750 and height of 483 so that the SVG image will have the same size as sammy.png. This will help in making the text look centered on the sammy.png image.


Next, you’ll draw the text graphics. Add the highlighted code to draw Sammy the Shark on the SVG container:


process_images/addTextOnImage.js
```
async function addTextOnImage() {
    ...
    const svg = `
    <svg width="${width}" height="${height}">
    <text x="50%" y="50%" text-anchor="middle" class="title">${text}</text>
    </svg>
    `;
  ....
}

```


The SVG text element has four attributes: x, y, text-anchor, and class. x and y define the position for the text you are drawing on the SVG container. The x attribute positions the text horizontally, and the y attribute positions the text vertically.


Setting x to 50% draws the text in the middle of the container on the x-axis, and setting y to 50% positions the text in the middle on y-axis of the SVG image.


The text-anchor aligns text horizontally. Setting text-anchor to middle will align the text on the center at the x coordinate you specified.


class defines a class name on the text element. You’ll use the class name to apply CSS styles to the text element.


${text} interpolates the string Sammy the Shark stored in the text variable. This is the text that will be drawn on the SVG image.


Next, add the highlighted code to style the text using CSS:


process_images/addTextOnImage.js
```
    const svg = `
    <svg width="${width}" height="${height}">
      <style>
      .title { fill: #001; font-size: 70px; font-weight: bold;}
      </style>
      <text x="50%" y="50%" text-anchor="middle" class="title">${text}</text>
    </svg>
    `;

```


In this code, fill changes the text color to black, font-size changes the font size, and font-weight changes the font weight.


At this point, you have written the code necessary to draw the text Sammy the Shark with SVG. Next, you’ll save the SVG image as a png with sharp so that you can see how SVG is drawing the text. Once that is done, you’ll composite the SVG image with sammy.png.


Add the highlighted code to save the SVG image as a png with sharp:


process_images/addTextOnImage.js
```
    ....
    const svgImage = `
    <svg width="${width}" height="${height}">
    ...
    </svg>
    `;
    const svgBuffer = Buffer.from(svgImage);
    const image = await sharp(svgBuffer).toFile("svg-image.png");
  } catch (error) {
    console.log(error);
  }
}

addTextOnImage();

```


Buffer.from() creates a Buffer object from the SVG image. A buffer is a temporary space in memory that stores binary data.


After creating the buffer object, you create a sharp instance with the buffer object as input. In addition to an image path, sharp also accepts a buffer, Uint9Array, or Uint8ClampedArray.


Finally, you save the SVG image in the project directory as svg-image.png.


Here is the complete code:


process_images/addTextOnImage.js
```
const sharp = require("sharp");

async function addTextOnImage() {
  try {
    const width = 750;
    const height = 483;
    const text = "Sammy the Shark";

    const svgImage = `
    <svg width="${width}" height="${height}">
      <style>
      .title { fill: #001; font-size: 70px; font-weight: bold;}
      </style>
      <text x="50%" y="50%" text-anchor="middle" class="title">${text}</text>
    </svg>
    `;
    const svgBuffer = Buffer.from(svgImage);
    const image = await sharp(svgBuffer).toFile("svg-image.png");
  } catch (error) {
    console.log(error);
  }
}

addTextOnImage()

```


Save and exit the file, then run your script with the following command:


```
node addTextOnImage.js

```



Note: If you installed Node.js using Option 2 — Installing Node.js with Apt Using a NodeSource PPA or Option 3 — Installing Node Using the Node Version Manager and getting the error fontconfig error: cannot load default config file: no such file: (null), install fontconfig to generate the font configuration file.
Update your server’s package index, and after that, use apt install to install fontconfig.
sudo apt update
sudo apt install fontconfig



Open svg-image.png on your local machine. You should now see the text Sammy the Shark rendered with a transparent background:





Now that you’ve confirmed the SVG code draws the text, you will composite the text graphics onto sammy.png.


Add the following highlighted code to composite the SVG text graphics image onto the sammy.png image.


process_images/addTextOnImage.js
```
const sharp = require("sharp");

async function addTextOnImage() {
  try {
    const width = 750;
    const height = 483;
    const text = "Sammy the Shark";

    const svgImage = `
    <svg width="${width}" height="${height}">
      <style>
      .title { fill: #001; font-size: 70px; font-weight: bold;}
      </style>
      <text x="50%" y="50%" text-anchor="middle" class="title">${text}</text>
    </svg>
    `;
    const svgBuffer = Buffer.from(svgImage);
    const image = await sharp("sammy.png")
      .composite([
        {
          input: svgBuffer,
          top: 0,
          left: 0,
        },
      ])
      .toFile("sammy-text-overlay.png");
  } catch (error) {
    console.log(error);
  }
}

addTextOnImage();

```


The composite() method reads the SVG image from the svgBuffer variable, and positions it 0 pixels from the top, and 0 pixels from the left edge of the sammy.png. Next, you save the composited image as sammy-text-overlay.png.


Save and close your file, then run your program using the following command:


```
node addTextOnImage.js


```


Open sammy-text-overlay.png on your local machine. You should see text added over the image:





You have now used the composite() method to add text created with SVG on another image.


# Conclusion


In this article, you learned how to use sharp methods to process images in Node.js. First, you created an instance to read an image and used the metadata() method to extract the image metadata. You then used the resize() method to resize an image. Afterwards, you used the format() method to change the image type, and compress the image. Next, you proceeded to use various sharp methods to crop, grayscale, rotate, and blur an image. Finally, you used the composite() method to composite an image, and add text on an image.


For more insight into additional sharp methods, visit the sharp documentation. If you want to continue learning Node.js, see How To Code in Node.js series.


