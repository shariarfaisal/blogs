# How To Build a Media Processing API in Node js With Express and FFmpeg wasm

```JavaScript``` ```HTML``` ```CSS``` ```Node.js``` ```API``` ```Development```

The author selected the Electronic Frontier Foundation to receive a donation as part of the Write for DOnations program.


## Introduction


Handling media assets is becoming a common requirement of modern back-end services. Using dedicated, cloud-based solutions may help when you’re dealing with massive scale or performing expensive operations, such as video transcoding. However, the extra cost and added complexity may be hard to justify when all you need is to extract a thumbnail from a video or check that user-generated content is in the correct format. Particularly at a smaller scale, it makes sense to add media processing capability directly to your Node.js API.


In this guide, you will build a media API in Node.js with Express and ffmpeg.wasm — a WebAssembly port of the popular media processing tool. You’ll build an endpoint that extracts a thumbnail from a video as an example. You can use the same techniques to add other features supported by FFmpeg to your API.


When you’re finished, you will have a good grasp on handling binary data in Express and processing them with ffmpeg.wasm. You’ll also handle requests made to your API that cannot be processed in parallel.


# Prerequisites


To complete this tutorial, you will need:


- A local development environment for Node.js. Follow How to Install Node.js and Create a Local Development Environment.
- Familiarity with building APIs with Express in Node.js. See How To Get Started with Node.js and Express.
- Experience with building websites with HTML and JavaScript. You can take a look at the tutorial How To Add JavaScript to HTML to review placing JavaScript in HTML.
- A video to download to test your implementation.

This tutorial was verified with Node v16.11.0, npm v7.15.1, express v4.17.1, and ffmpeg.wasm v0.10.1.


# Step 1 — Setting Up the Project and Creating a Basic Express Server


In this step, you will create a project directory, initialize Node.js and install ffmpeg, and set up a basic Express server.


Start by opening the terminal and creating a new directory for the project:


```
mkdir ffmpeg-api


```


Navigate to the new directory:


```
cd ffmpeg-api


```


Use npm init to create a new package.json file. The -y parameter indicates that you’re happy with the default settings for the project.


```
npm init -y


```


Finally, use npm install to install the packages required to build the API. The --save flag indicates that you wish to save those as dependencies in the package.json file.


```
npm install --save @ffmpeg/ffmpeg @ffmpeg/core express cors multer p-queue


```


Now that you have installed ffmpeg, you’ll set up a web server that responds to requests using Express.


First, open a new file called server.mjs with nano or your editor of choice:


```
nano server.mjs


```


The code in this file will register the cors middleware which will permit requests made from websites with a different origin. At the top of the file, import the express and cors dependencies:


server.mjs
```
import express from 'express';
import cors from 'cors';

```


Then, create an Express app and start the server on the port :3000 by adding the following code below the import statements:


server.mjs
```
...
const app = express();
const port = 3000;

app.use(cors());

app.listen(port, () => {
    console.log(`[info] ffmpeg-api listening at http://localhost:${port}`)
});

```


You can start the server by running the following command:


```
node server.mjs


```


You’ll see the following output:


```
Output[info] ffmpeg-api listening at http://localhost:3000

```


When you try loading http://localhost:3000 in your browser, you’ll see Cannot GET /. This is Express telling you it is listening for requests.


With your Express server now set up, you’ll create a client to upload the video and make requests to your Express server.


# Step 2 — Creating a Client and Testing the Server


In this section, you’ll create a web page that will let you select a file and upload it to the API for processing.


Start by opening a new file called client.html:


```
nano client.html


```


In your client.html file, create a file input and a Create Thumbnail button. Below, add an empty <div> element to display errors and an image that will show the thumbnail that the API sends back. At the very end of the <body> tag, load a script called client.js. Your final HTML template should look as follows:


client.html
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Create a Thumbnail from a Video</title>
    <style>
        #thumbnail {
            max-width: 100%;
        }
    </style>
</head>
<body>
    <div>
        <input id="file-input" type="file" />
        <button id="submit">Create Thumbnail</button>
        <div id="error"></div>
        <img id="thumbnail" />
    </div>
    <script src="client.js"></script>
</body>
</html>

```


Note that each element has a unique id. You’ll need them when referring to the elements from the client.js script. The styling on the #thumbnail element is there to ensure that the image fits on the screen when it loads.


Save the client.html file and open client.js:


```
nano client.js


```


In your client.js file, start by defining variables that store references to your HTML elements you created:


client.js
```
const fileInput = document.querySelector('#file-input');
const submitButton = document.querySelector('#submit');
const thumbnailPreview = document.querySelector('#thumbnail');
const errorDiv = document.querySelector('#error');

```


Then, attach a click event listener to the submitButton variable to check whether you’ve selected a file:


client.js
```
...
submitButton.addEventListener('click', async () => {
    const { files } = fileInput;
}

```


Next, create a function showError() that will output an error message when a file is not selected. Add the showError() function above your event listener:


client.js
```
const fileInput = document.querySelector('#file-input');
const submitButton = document.querySelector('#submit');
const thumbnailPreview = document.querySelector('#thumbnail');
const errorDiv = document.querySelector('#error');

function showError(msg) {
    errorDiv.innerText = `ERROR: ${msg}`;
}

submitButton.addEventListener('click', async () => {
...

```


Now, you will build a function createThumbnail() that will make a request to the API, send the video, and receive a thumbnail in response. At the top of your client.js file, define a new constant with the URL to a /thumbnail endpoint:


```
const API_ENDPOINT = 'http://localhost:3000/thumbnail';

const fileInput = document.querySelector('#file-input');
const submitButton = document.querySelector('#submit');
const thumbnailPreview = document.querySelector('#thumbnail');
const errorDiv = document.querySelector('#error');
...

```


You will define and use the /thumbnail endpoint in your Express server.


Next, add the createThumbnail() function below your showError() function:


client.js
```
...
function showError(msg) {
    errorDiv.innerText = `ERROR: ${msg}`;
}

async function createThumbnail(video) {

}
...

```


Web APIs frequently use JSON to transfer structured data from and to the client. To include a video in a JSON, you would have to encode it in base64, which would increase its size by about 30%. You can avoid this by using multipart requests instead. Multipart requests allow you to transfer structured data including binary files over http without the unnecessary overhead. You can do this using the FormData() constructor function.


Inside the createThumbnail() function, create an instance of FormData and append the video file to the object. Then make a POST request to the API endpoint using the Fetch API with the FormData() instance as the body. Interpret the response as a binary file (or blob) and convert it to a data URL so that you can assign it to the <img> tag you created earlier.


Here’s the full implementation of createThumbnail():


client.js
```
...
async function createThumbnail(video) {
    const payload = new FormData();
    payload.append('video', video);

    const res = await fetch(API_ENDPOINT, {
        method: 'POST',
        body: payload
    });

    if (!res.ok) {
        throw new Error('Creating thumbnail failed');
    }

    const thumbnailBlob = await res.blob();
    const thumbnail = await blobToDataURL(thumbnailBlob);

    return thumbnail;
}
...

```


You’ll notice createThumbnail() has the function blobToDataURL() in its body. This is a helper function that will convert a blob to a data URL.


Above your createThumbnail() function, create the function blobDataToURL() that returns a promise:


client.js
```
...
async function blobToDataURL(blob) {
    return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.onload = () => resolve(reader.result);
        reader.onerror = () => reject(reader.error);
        reader.onabort = () => reject(new Error("Read aborted"));
        reader.readAsDataURL(blob);
    });
}
...

```


blobToDataURL() uses FileReader to read the contents of the binary file and format it as a data URL.


With the createThumbnail() and showError() functions now defined, you can use them to finish implementing the event listener:


client.js
```
...
submitButton.addEventListener('click', async () => {
    const { files } = fileInput;

    if (files.length > 0) {
        const file = files[0];
        try {
            const thumbnail = await createThumbnail(file);
            thumbnailPreview.src = thumbnail;
        } catch(error) {
            showError(error);
        }
    } else {
        showError('Please select a file');
    }
});

```


When a user clicks on the button, the event listener will pass the file to the createThumbnail() function. If successful, it will assign the thumbnail to the <img> element you created earlier. In case the user doesn’t select a file or the request fails, it will call the showError() function to display an error.


At this point, your client.js file will look like the following:


client.js
```
const API_ENDPOINT = 'http://localhost:3000/thumbnail';

const fileInput = document.querySelector('#file-input');
const submitButton = document.querySelector('#submit');
const thumbnailPreview = document.querySelector('#thumbnail');
const errorDiv = document.querySelector('#error');

function showError(msg) {
    errorDiv.innerText = `ERROR: ${msg}`;
}

async function blobToDataURL(blob) {
    return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.onload = () => resolve(reader.result);
        reader.onerror = () => reject(reader.error);
        reader.onabort = () => reject(new Error("Read aborted"));
        reader.readAsDataURL(blob);
    });
}

async function createThumbnail(video) {
    const payload = new FormData();
    payload.append('video', video);

    const res = await fetch(API_ENDPOINT, {
        method: 'POST',
        body: payload
    });

    if (!res.ok) {
        throw new Error('Creating thumbnail failed');
    }

    const thumbnailBlob = await res.blob();
    const thumbnail = await blobToDataURL(thumbnailBlob);

    return thumbnail;
}

submitButton.addEventListener('click', async () => {
    const { files } = fileInput;

    if (files.length > 0) {
        const file = files[0];

        try {
            const thumbnail = await createThumbnail(file);
            thumbnailPreview.src = thumbnail;
        } catch(error) {
            showError(error);
        }
    } else {
        showError('Please select a file');
    }
});

```


Start the server again by running:


```
node server.mjs


```


With your client now set up, uploading the video file here will result in receiving an error message. This is because the /thumbnail endpoint is not built yet. In the next step, you’ll create the /thumbnail endpoint in Express to accept the video file and create the thumbnail.


# Step 3 — Setting Up an Endpoint to Accept Binary Data


In this step, you will set up a POST request for the /thumbnail endpoint and use middleware to accept multipart requests.


Open server.mjs in an editor:


```
nano server.mjs


```


Then, import multer at the top of the file:


server.mjs
```
import express from 'express';
import cors from 'cors';
import multer from 'multer';
...

```


Multer is a middleware that processes incoming multipart/form-data requests before passing them to your endpoint handler. It extracts fields and files from the body and makes them available as an array on the request object in Express. You can configure where to store the uploaded files and set limits on file size and format.


After importing it, initialize the multer middleware with the following options:


server.mjs
```
...
const app = express();
const port = 3000;

const upload = multer({
    storage: multer.memoryStorage(),
    limits: { fileSize: 100 * 1024 * 1024 }
});

app.use(cors());
...

```


The storage option lets you choose where to store the incoming files. Calling multer.memoryStorage() will initialize a storage engine that keeps files in Buffer objects in memory as opposed to writing them to disk. The limits option lets you define various limits on what files will be accepted. Set the fileSize limit to 100MB or a different number that matches your needs and the amount of memory available on your server. This will prevent your API from crashing when the input file is too big.



Note: Due to the limitations of WebAssembly, ffmpeg.wasm cannot handle input files over 2GB in size.

Next, set up the POST /thumbnail endpoint itself:


server.mjs
```
...
app.use(cors());

app.post('/thumbnail', upload.single('video'), async (req, res) => {
    const videoData = req.file.buffer;

    res.sendStatus(200);
});

app.listen(port, () => {
    console.log(`[info] ffmpeg-api listening at http://localhost:${port}`)
});

```


The upload.single('video') call will set up a middleware for that endpoint only that will parse the body of a multipart request that includes a single file. The first parameter is the field name. It must match the one you gave to FormData when creating the request in client.js. In this case, it’s video. multer will then attach the parsed file to the req parameter. The content of the file will be under req.file.buffer.


At this point, the endpoint doesn’t do anything with the data it receives. It acknowledges the request by sending an empty 200 response. In the next step, you’ll replace that with the code that extracts a thumbnail from the video data received.


# Step 4 — Processing Media With ffmpeg.wasm


In this step, you’ll use ffmpeg.wasm to extract a thumbnail from the video file received by the POST /thumbnail endpoint.


ffmpeg.wasm is a pure WebAssembly and JavaScript port of FFmpeg. Its main goal is to allow running FFmpeg directly in the browser. However, because Node.js is built on top of V8 — Chrome’s JavaScript engine — you can use the library on the server too.


The benefit of using a native port of FFmpeg over a wrapper built on top of the ffmpeg command is that if you’re planning to deploy your app with Docker, you don’t have to build a custom image that includes both FFmpeg and Node.js. This will save you time and reduce the maintenance burden of your service.


Add the following import to the top of server.mjs:


server.mjs
```
import express from 'express';
import cors from 'cors';
import multer from 'multer';
import { createFFmpeg } from '@ffmpeg/ffmpeg';
...

```


Then, create an instance of ffmpeg.wasm and start loading the core:


server.mjs
```
...
import { createFFmpeg } from '@ffmpeg/ffmpeg';

const ffmpegInstance = createFFmpeg({ log: true });
let ffmpegLoadingPromise = ffmpegInstance.load();

const app = express();
...

```


The ffmpegInstance variable holds a reference to the library. Calling ffmpegInstance.load() starts loading the core into memory asynchronously and returns a promise. Store the promise in the ffmpegLoadingPromise variable so that you can check whether the core has loaded.


Next, define the following helper function that will use fmpegLoadingPromise to wait for the core to load in case the first request arrives before it’s ready:


server.mjs
```
...
let ffmpegLoadingPromise = ffmpegInstance.load();

async function getFFmpeg() {
    if (ffmpegLoadingPromise) {
        await ffmpegLoadingPromise;
        ffmpegLoadingPromise = undefined;
    }

    return ffmpegInstance;
}

const app = express();
...

```


The getFFmpeg() function returns a reference to the library stored in the ffmpegInstance variable. Before returning it, it checks whether the library has finished loading. If not, it will wait until ffmpegLoadingPromise resolves. In case the first request to your POST /thumbnail endpoint arrives before ffmpegInstance is ready to use, your API will wait and resolve it when it can rather than rejecting it.


Now, implement the POST /thumbnail endpoint handler. Replace res.sendStatus(200); at the end of the end of the function with a call to getFFmpeg to get a reference to ffmpeg.wasm when it’s ready:


server.mjs
```
...
app.post('/thumbnail', upload.single('video'), async (req, res) => {
    const videoData = req.file.buffer;

    const ffmpeg = await getFFmpeg();
});
...

```


ffmpeg.wasm works on top of an in-memory file system. You can read and write to it using ffmpeg.FS. When running FFmpeg operations, you will pass virtual file names to the ffmpeg.run function as an argument the same way as you would when working with the CLI tool. Any output files created by FFmpeg will be written to the file system for you to retrieve.


In this case, the input file is a video. The output file will be a single PNG image. Define the following variables:


server.mjs
```
...
    const ffmpeg = await getFFmpeg();

    const inputFileName = `input-video`;
    const outputFileName = `output-image.png`;
    let outputData = null;
});
...

```


The file names will be used on the virtual file system. outputData is where you’ll store the thumbnail when it’s ready.


Call ffmpeg.FS() to write the video data to the in-memory file system:


server.mjs
```
...
    let outputData = null;

    ffmpeg.FS('writeFile', inputFileName, videoData);
});
...

```


Then, run the FFmpeg operation:


server.mjs
```
...
    ffmpeg.FS('writeFile', inputFileName, videoData);

    await ffmpeg.run(
        '-ss', '00:00:01.000',
        '-i', inputFileName,
        '-frames:v', '1',
        outputFileName
    );
});
...

```


The -i parameter specifies the input file. -ss seeks to the specified time (in this case, 1 second from the beginning of the video). -frames:v limits the number of frames that will be written to the output (a single frame in this scenario). outputFileName at the end indicates where will FFmpeg write the output.


After FFmpeg exits, use ffmpeg.FS() to read the data from the file system and delete both the input and output files to free up memory:


server.mjs
```
...
    await ffmpeg.run(
        '-ss', '00:00:01.000',
        '-i', inputFileName,
        '-frames:v', '1',
        outputFileName
    );

    outputData = ffmpeg.FS('readFile', outputFileName);
    ffmpeg.FS('unlink', inputFileName);
    ffmpeg.FS('unlink', outputFileName);
});
...

```


Finally, dispatch the output data in the body of the response:


server.mjs
```
...
    ffmpeg.FS('unlink', outputFileName);

    res.writeHead(200, {
        'Content-Type': 'image/png',
        'Content-Disposition': `attachment;filename=${outputFileName}`,
        'Content-Length': outputData.length
    });
    res.end(Buffer.from(outputData, 'binary'));
});
...

```


Calling res.writeHead() dispatches the response head. The second parameter includes custom http headers) with information about the data in the body of the request that will follow. The res.end() function sends the data from its first argument as the body of the request and finalizes the request. The outputData variable is a raw array of bytes as returned by ffmpeg.FS(). Passing it to Buffer.from() initializes a Buffer to ensure the binary data will be handled correctly by res.end().


At this point, your POST /thumbnail endpoint implementation should look like this:


server.mjs
```
...
app.post('/thumbnail', upload.single('video'), async (req, res) => {
    const videoData = req.file.buffer;

    const ffmpeg = await getFFmpeg();

    const inputFileName = `input-video`;
    const outputFileName = `output-image.png`;
    let outputData = null;

    ffmpeg.FS('writeFile', inputFileName, videoData);

    await ffmpeg.run(
        '-ss', '00:00:01.000',
        '-i', inputFileName,
        '-frames:v', '1',
        outputFileName
    );

    outputData = ffmpeg.FS('readFile', outputFileName);
    ffmpeg.FS('unlink', inputFileName);
    ffmpeg.FS('unlink', outputFileName);

    res.writeHead(200, {
        'Content-Type': 'image/png',
        'Content-Disposition': `attachment;filename=${outputFileName}`,
        'Content-Length': outputData.length
    });
    res.end(Buffer.from(outputData, 'binary'));
});
...

```


Aside from the 100MB file limit for uploads, there’s no input validation or error handling. When ffmpeg.wasm fails to process a file, reading the output from the virtual file system will fail and prevent the response from being sent. For the purposes of this tutorial, wrap the implementation of the endpoint in a try-catch block to handle that scenario:


server.mjs
```
...
app.post('/thumbnail', upload.single('video'), async (req, res) => {
    try {
        const videoData = req.file.buffer;

        const ffmpeg = await getFFmpeg();

        const inputFileName = `input-video`;
        const outputFileName = `output-image.png`;
        let outputData = null;

        ffmpeg.FS('writeFile', inputFileName, videoData);

        await ffmpeg.run(
            '-ss', '00:00:01.000',
            '-i', inputFileName,
            '-frames:v', '1',
            outputFileName
        );

        outputData = ffmpeg.FS('readFile', outputFileName);
        ffmpeg.FS('unlink', inputFileName);
        ffmpeg.FS('unlink', outputFileName);

        res.writeHead(200, {
            'Content-Type': 'image/png',
            'Content-Disposition': `attachment;filename=${outputFileName}`,
            'Content-Length': outputData.length
        });
        res.end(Buffer.from(outputData, 'binary'));
    } catch(error) {
        console.error(error);
        res.sendStatus(500);
    }
...
});

```


Secondly, ffmpeg.wasm cannot handle two requests in parallel. You can try this yourself by launching the server:


```
node --experimental-wasm-threads server.mjs


```


Note the flag required for ffmpeg.wasm to work. The library depends on WebAssembly threads and bulk memory operations. These have been in V8/Chrome since 2019. However, as of Node.js v16.11.0, WebAssembly threads remain behind a flag in case there might be changes before the proposal is finalised. Bulk memory operations also require a flag in older versions of Node. If you’re running Node.js 15 or lower, add --experimental-wasm-bulk-memory as well.


The output of the command will look like this:


```
Output[info] use ffmpeg.wasm v0.10.1
[info] load ffmpeg-core
[info] loading ffmpeg-core
[info] fetch ffmpeg.wasm-core script from @ffmpeg/core
[info] ffmpeg-api listening at http://localhost:3000
[info] ffmpeg-core loaded

```


Open client.html in a web browser and select a video file. When you click the Create Thumbnail button, you should see the thumbnail appear on the page. Behind the scenes, the site uploads the video to the API, which processes it and responds with the image. However, when you click the button repeatedly in quick succession, the API will handle the first request. The subsequent requests will fail:


```
OutputError: ffmpeg.wasm can only run one command at a time
    at Object.run (.../ffmpeg-api/node_modules/@ffmpeg/ffmpeg/src/createFFmpeg.js:126:13)
    at file://.../ffmpeg-api/server.mjs:54:26
    at runMicrotasks (<anonymous>)
    at processTicksAndRejections (internal/process/task_queues.js:95:5)

```


In the next section, you’ll learn how to deal with concurrent requests.


# Step 5 — Handling Concurrent Requests


Since ffmpeg.wasm can only execute a single operation at a time, you’ll need a way of serializing requests that come in and processing them one at a time. In this scenario, a promise queue is a perfect solution. Instead of starting to process each request right away, it will be queued up and processed when all the requests that arrived before it have been handled.


Open server.mjs in your preferred editor:


```
nano server.mjs


```


Import p-queue at the top of server.mjs:


server.mjs
```
import express from 'express';
import cors from 'cors';
import { createFFmpeg } from '@ffmpeg/ffmpeg';
import PQueue from 'p-queue';
...

```


Then, create a new queue at the top of server.mjs file under the variable ffmpegLoadingPromise:


server.mjs
```
...
const ffmpegInstance = createFFmpeg({ log: true });
let ffmpegLoadingPromise = ffmpegInstance.load();

const requestQueue = new PQueue({ concurrency: 1 });
...

```


In the POST /thumbnail endpoint handler, wrap the calls to ffmpeg in a function that will be queued up:


server.mjs
```
...
app.post('/thumbnail', upload.single('video'), async (req, res) => {
    try {
        const videoData = req.file.buffer;

        const ffmpeg = await getFFmpeg();

        const inputFileName = `input-video`;
        const outputFileName = `thumbnail.png`;
        let outputData = null;

        await requestQueue.add(async () => {
            ffmpeg.FS('writeFile', inputFileName, videoData);

            await ffmpeg.run(
                '-ss', '00:00:01.000',
                '-i', inputFileName,
                '-frames:v', '1',
                outputFileName
            );

            outputData = ffmpeg.FS('readFile', outputFileName);
            ffmpeg.FS('unlink', inputFileName);
            ffmpeg.FS('unlink', outputFileName);
        });

        res.writeHead(200, {
            'Content-Type': 'image/png',
            'Content-Disposition': `attachment;filename=${outputFileName}`,
            'Content-Length': outputData.length
        });
        res.end(Buffer.from(outputData, 'binary'));
    } catch(error) {
        console.error(error);
        res.sendStatus(500);
    }
});
...

```


Every time a new request comes in, it will only start processing when there’s nothing else queued up in front of it. Note that the final sending of the response can happen asynchronously. Once the ffmpeg.wasm operation finishes running, another request can start processing while the response goes out.


To test that everything works as expected, start up the server again:


```
node --experimental-wasm-threads server.mjs


```


Open the client.html file in your browser and try uploading a file.





With the queue in place, the API will now respond every time. The requests will be handled sequentially in the order in which they arrive.


# Conclusion


In this article, you built a Node.js service that extracts a thumbnail from a video using ffmpeg.wasm. You learned how to upload binary data from the browser to your Express API using multipart requests and how to process media with FFmpeg in Node.js without relying on external tools or having to write data to disk.


FFmpeg is an incredibly versatile tool. You can use the knowledge from this tutorial to take advantage of any features that FFmpeg supports and use them in your project. For example, to generate a three-second GIF, change the ffmpeg.run call to this on the POST /thumbnail endpoint:


server.mjs
```
...
await ffmpeg.run(
    '-y',
    '-t', '3',
    '-i', inputFileName,
    '-filter_complex', 'fps=5,scale=720:-1:flags=lanczos[x];[x]split[x1][x2];[x1]palettegen[p];[x2][p]paletteuse',
    '-f', 'gif',
    outputFileName
);
...

```


The library accepts the same parameters as the original ffmpeg CLI tool. You can use the official documentation to find a solution for your use case and test it quickly in the terminal.


Thanks to ffmpeg.wasm being self-contained, you can dockerize this service using the stock Node.js base images and scale your service up by keeping multiple nodes behind a load balancer. Follow the tutorial How To Build a Node.js Application with Docker to learn more.


If your use case requires performing more expensive operations, such as transcoding large videos, make sure that you run your service on machines with enough memory to store them. Due to current limitations in WebAssembly, the maximum input file size cannot exceed 2GB, although this might change in the future.


Additionally, ffmpeg.wasm cannot take advantage of some x86 assembly optimizations from the original FFmpeg codebase. That means some operations can take a long time to finish. If that’s the case, consider whether this is the right solution for your use case. Alternatively, make requests to your API asynchronous. Instead of waiting for the operation to finish, queue it up and respond with a unique ID. Create another endpoint that the clients can query to find out whether the processing ended and the output file is ready. Learn more about the asynchronous request-reply pattern for REST APIs and how to implement it.


