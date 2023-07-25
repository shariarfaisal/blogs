# Using Buffers in Node js

```JavaScript``` ```Development``` ```Node.js``` ```Networking```

The author selected the COVID-19 Relief Fund to receive a donation as part of the Write for DOnations program.


## Introduction


A buffer is a space in memory (typically RAM) that stores binary data. In Node.js, we can access these spaces of memory with the built-in Buffer class. Buffers store a sequence of integers, similar to an array in JavaScript. Unlike arrays, you cannot change the size of a buffer once it is created.


You may have used buffers implicitly if you wrote Node.js code already. For example, when you read from a file with fs.readFile(), the data returned to the callback or Promise is a buffer object. Additionally, when HTTP requests are made in Node.js, they return data streams that are temporarily stored in an internal buffer when the client cannot process the stream all at once.


Buffers are useful when you’re interacting with binary data, usually at lower networking levels. They also equip you with the ability to do fine-grained data manipulation in Node.js.


In this tutorial, you will use the Node.js REPL to run through various examples of buffers, such as creating buffers, reading from buffers, writing to and copying from buffers, and using buffers to convert between binary and encoded data. By the end of the tutorial, you’ll have learned how to use the Buffer class to work with binary data.


# Prerequisites


- You will need Node.js installed on your development machine. This tutorial uses version 10.19.0. To install this on macOS or Ubuntu 18.04, follow the steps in How To Install Node.js and Create a Local Development Environment on macOS or the Installing Using a PPA section of How To Install Node.js on Ubuntu 18.04.
- In this tutorial, you will interact with buffers in the Node.js REPL (Read-Evaluate-Print-Loop). If you want a refresher on how to use the Node.js REPL effectively, you can read our guide on How To Use the Node.js REPL.
- For this article we expect the user to be comfortable with basic JavaScript and its data types. You can learn those fundamentals with our How To Code in JavaScript series.

# Step 1 — Creating a Buffer


This first step will show you the two primary ways to create a buffer object in Node.js.


To decide what method to use, you need to answer this question: Do you want to create a new buffer or extract a buffer from existing data? If you are going to store data in memory that you have yet to receive, you’ll want to create a new buffer. In Node.js we use the alloc() function of the Buffer class to do this.


Let’s open the Node.js REPL to see for ourselves. In your terminal, enter the node command:


```
node


```


You will see the prompt begin with >.


The alloc() function takes the size of the buffer as its first and only required argument. The size is an integer representing how many bytes of memory the buffer object will use. For example, if we wanted to create a buffer that was 1KB (kilobyte) large, equivalent to 1024 bytes, we would enter this in the console:


```
const firstBuf = Buffer.alloc(1024);


```


To create a new buffer, we used the globally available Buffer class, which has the alloc() method. By providing 1024 as the argument for alloc(), we created a buffer that’s 1KB large.


By default, when you initialize a buffer with alloc(), the buffer is filled with binary zeroes as a placeholder for later data. However, we can change the default value if we’d like to. If we wanted to create a new buffer with 1s instead of 0s, we would set the alloc() function’s second parameter—fill.


In your terminal, create a new buffer at the REPL prompt that’s filled with 1s:


```
const filledBuf = Buffer.alloc(1024, 1);


```


We just created a new buffer object that references a space in memory that stores 1KB of 1s. Although we entered an integer, all data stored in a buffer is binary data.


Binary data can come in many different formats. For example, let’s consider a binary sequence representing a byte of data: 01110110. If this binary sequence represented a string in English using the ASCII encoding standard, it would be the letter v. However, if our computer was processing an image, that binary sequence could contain information about the color of a pixel.


The computer knows to process them differently because the bytes are encoded differently. Byte encoding is the format of the byte. A buffer in Node.js uses the UTF-8 encoding scheme by default if it’s initialized with string data. A byte in UTF-8 represents a number, a letter (in English and in other languages), or a symbol. UTF-8 is a superset of ASCII, the American Standard Code for Information Interchange. ASCII can encode bytes with uppercase and lowercase English letters, the numbers 0-9, and a few other symbols like the exclamation mark (!) or the ampersand sign (&).


If we were writing a program that could only work with ASCII characters, we could change the encoding used by our buffer with the alloc() function’s third argument—encoding.


Let’s create a new buffer that’s five bytes long and stores only ASCII characters:


```
const asciiBuf = Buffer.alloc(5, 'a', 'ascii');


```


The buffer is initialized with five bytes of the character a, using the ASCII representation.



Note: By default, Node.js supports the following character encodings:

ASCII, represented as ascii
UTF-8, represented as utf-8 or utf8
UTF-16, represented as utf-16le or utf16le
UCS-2, represented as ucs-2 or ucs2
Base64, represented as base64
Hexadecimal, represented as hex
ISO/IEC 8859-1, represented as latin1 or binary

All of these values can be used in Buffer class functions that accept an encoding parameter. Therefore, these values are all valid for the alloc() method.

So far we’ve been creating new buffers with the alloc() function. But sometimes we may want to create a buffer from data that already exists, like a string or array.


To create a buffer from pre-existing data, we use the from() method. We can use that function to create buffers from:


- An array of integers: The integer values can be between 0 and 255.
- An ArrayBuffer: This is a JavaScript object that stores a fixed length of bytes.
- A string.
- Another buffer.
- Other JavaScript objects that have a Symbol.toPrimitive property. That property tells JavaScript how to convert the object to a primitive data type: boolean, null, undefined, number, string, or symbol. You can read more about Symbols at Mozilla’s JavaScript documentation.

Let’s see how we can create a buffer from a string. In the Node.js prompt, enter this:


```
const stringBuf = Buffer.from('My name is Paul');


```


We now have a buffer object created from the string My name is Paul. Let’s create a new buffer from another buffer we made earlier:


```
const asciiCopy = Buffer.from(asciiBuf);


```


We’ve now created a new buffer asciiCopy that contains the same data as asciiBuf.


Now that we have experienced creating buffers, we can dive into examples of reading their data.


# Step 2 — Reading from a Buffer


There are many ways to access data in a Buffer. We can access an individual byte in a buffer or we can extract the entire contents.


To access one byte of a buffer, we pass the index or location of the byte we want. Buffers store data sequentially like arrays. They also index their data like arrays, starting at 0. We can use array notation on the buffer object to get an individual byte.


Let’s see how this looks by creating a buffer from a string in the REPL:


```
const hiBuf = Buffer.from('Hi!');


```


Now let’s read the first byte of the buffer:


```
hiBuf[0];


```


As you press ENTER, the REPL will display:


```
Output72

```


The integer 72 corresponds the UTF-8 representation for the letter H.



Note: The values for bytes can be numbers between 0 and 255. A byte is a sequence of 8 bits. A bit is binary, and therefore can only have one of two values: 0 or 1. If we have a sequence of 8 bits and two possible values per bit, then we have a maximum of 2⁸ possible values for a byte. That works out to a maximum of 256 values. Since we start counting from zero, that means our highest number is 255.

Let’s do the same for the second byte. Enter the following in the REPL:


```
hiBuf[1];


```


The REPL returns 105, which represents the lowercase i.


Finally, let’s get the third character:


```
hiBuf[2];


```


You will see 33 displayed in the REPL, which corresponds to !.


Let’s try to retrieve a byte from an invalid index:


```
hiBuf[3];


```


The REPL will return:


```
Outputundefined

```


This is just like if we tried to access an element in an array with an incorrect index.


Now that we’ve seen how to read individual bytes of a buffer, let’s see our options for retrieving all the data stored in a buffer at once. The buffer object comes with the toString() and the toJSON() methods, which return the entire contents of a buffer in two different formats.


As its name suggests, the toString() method converts the bytes of the buffer into a string and returns it to the user. If we use this method on hiBuf, we will get the string Hi!. Let’s try it!


In the prompt, enter:


```
hiBuf.toString();


```


The REPL will return:


```
Output'Hi!'

```


That buffer was created from a string. Let’s see what happens if we use the toString() on a buffer that was not made from string data.


Let’s create a new, empty buffer that’s 10 bytes large:


```
const tenZeroes = Buffer.alloc(10);


```


Now, let’s use the toString() method:


```
tenZeroes.toString();


```


We will see the following result:


```
'\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000\u0000'

```


The string \u0000 is the Unicode character for NULL. It corresponds to the number 0. When the buffer’s data is not encoded as a string, the toString() method returns the UTF-8 encoding of the bytes.


The toString() has an optional parameter, encoding. We can use this parameter to change the encoding of the buffer data that’s returned.


For example, if you wanted the hexadecimal encoding for hiBuf you would enter the following at the prompt:


```
hiBuf.toString('hex');


```


That statement will evaluate to:


```
Output'486921'

```


486921 is the hexadecimal representation for the bytes that represent the string Hi!. In Node.js, when users want to convert the encoding of data from one form to another, they usually put the string in a buffer and call toString() with their desired encoding.


The toJSON() method behaves differently. Regardless of whether the buffer was made from a string or not, it always returns the data as the integer representation of the byte.


Let’s re-use the hiBuf and tenZeroes buffers to practice using toJSON(). At the prompt, enter:


```
hiBuf.toJSON();


```


The REPL will return:


```
Output{ type: 'Buffer', data: [ 72, 105, 33 ] }

```


The JSON object has a type property that will always be Buffer. That’s so programs can distinguish these JSON object from other JSON objects.


The data property contains an array of the integer representation of the bytes. You may have noticed that 72, 105, and 33 correspond to the values we received when we individually pulled the bytes.


Let’s try the toJSON() method with tenZeroes:


```
tenZeroes.toJSON();


```


In the REPL you will see the following:


```
Output{ type: 'Buffer', data: [
    0, 0, 0, 0, 0,
    0, 0, 0, 0, 0
  ] }

```


The type is the same as noted before. However, the data is now an array with ten zeroes.


Now that we’ve covered the main ways to read from a buffer, let’s look at how we modify a buffer’s contents.


# Step 3 — Modifying a Buffer


There are many ways we can modify an existing buffer object. Similar to reading, we can modify buffer bytes individually using the array syntax. We can also write new contents to a buffer, replacing the existing data.


Let’s begin by looking at how we can change individual bytes of a buffer. Recall our buffer variable hiBuf, which contains the string Hi!. Let’s change each byte so that it contains Hey instead.


In the REPL, let’s first try setting the second element of hiBuf to e:


```
hiBuf[1] = 'e';


```


Now, let’s see this buffer as a string to confirm it’s storing the right data. Follow up by calling the toString() method:


```
hiBuf.toString();


```


It will be evaluated as:


```
Output'H\u0000!'

```


We received that strange output because the buffer can only accept an integer value. We can’t assign it to the letter e; rather, we have to assign it the number whose binary equivalent represents e:


```
hiBuf[1] = 101;


```


Now when we call the toString() method:


```
hiBuf.toString();


```


We get this output in the REPL:


```
Output'He!'

```


To change the last character in the buffer, we need to set the third element to the integer that corresponds to the byte for y:


```
hiBuf[2] = 121;


```


Let’s confirm by using the toString() method once again:


```
hiBuf.toString();


```


Your REPL will display:


```
Output'Hey'

```


If we try to write a byte that’s outside the range of the buffer, it will be ignored and the contents of the buffer won’t change. For example, let’s try to set the non-existent fourth element of the buffer to o:


```
hiBuf[3] = 111;


```


We can confirm that the buffer is unchanged with the toString() method:


```
hiBuf.toString();


```


The output is still:


```
Output'Hey'

```


If we wanted to change the contents of the entire buffer, we can use the write() method. The write() method accepts a string that will replace the contents of a buffer.


Let’s use the write() method to change the contents of hiBuf back to Hi!. In your Node.js shell, type the following command at the prompt:


```
hiBuf.write('Hi!');


```


The write() method returned 3 in the REPL. This is because it wrote three bytes of data. Each letter has one byte in size, since this buffer uses UTF-8 encoding, which uses a byte for each character. If the buffer used UTF-16 encoding, which has a minimum of two bytes per character, then the write() function would have returned 6.


Now verify the contents of the buffer by using toString():


```
hiBuf.toString();


```


The REPL will produce:


```
Output'Hi!'

```


This is quicker than having to change each element byte-by-byte.


If you try to write more bytes than a buffer’s size, the buffer object will only accept what bytes fit. To illustrate, let’s create a buffer that stores three bytes:


```
const petBuf = Buffer.alloc(3);


```


Now let’s attempt to write Cats to it:


```
petBuf.write('Cats');


```


When the write() call is evaluated, the REPL returns 3 indicating only three bytes were written to the buffer. Now confirm that the buffer contains the first three bytes:


```
petBuf.toString();


```


The REPL returns:


```
Output'Cat'

```


The write() function adds the bytes in sequential order, so only the first three bytes were placed in the buffer.


By contrast, let’s make a Buffer that stores four bytes:


```
const petBuf2 = Buffer.alloc(4);


```


Write the same contents to it:


```
petBuf2.write('Cats');


```


Then add some new content that occupies less space than the original content:


```
petBuf2.write('Hi');


```


Since buffers write sequentially, starting from 0, if we print the buffer’s contents:


```
petBuf2.toString();


```


We’d be greeted with:


```
Output'Hits'

```


The first two characters are overwritten, but the rest of the buffer is untouched.


Sometimes the data we want in our pre-existing buffer is not in a string but resides in another buffer object. In these cases, we can use the copy() function to modify what our buffer is storing.


Let’s create two new buffers:


```
const wordsBuf = Buffer.from('Banana Nananana');
const catchphraseBuf = Buffer.from('Not sure Turtle!');


```


The wordsBuf and catchphraseBuf buffers both contain string data. We want to modify catchphraseBuf so that it stores Nananana Turtle! instead of Not sure Turtle!. We’ll use copy() to get Nananana from wordsBuf to catchphraseBuf.


To copy data from one buffer to the other, we’ll use the copy() method on the buffer that’s the source of the information. Therefore, as wordsBuf has the string data we want to copy, we need to copy like this:


```
wordsBuf.copy(catchphraseBuf);


```


The target parameter in this case is the catchphraseBuf buffer.


When we enter that into the REPL, it returns 15 indicating that 15 bytes were written. The string Nananana only uses 8 bytes of data, so we immediately know that our copy did not go as intended. Use the toString() method to see the contents of catchphraseBuf:


```
catchphraseBuf.toString();


```


The REPL returns:


```
Output'Banana Nananana!'

```


By default, copy() took the entire contents of wordsBuf and placed it into catchphraseBuf. We need to be more selective for our goal and only copy Nananana. Let’s re-write the original contents of catchphraseBuf before continuing:


```
catchphraseBuf.write('Not sure Turtle!');


```


The copy() function has a few more parameters that allow us to customize what data is copied to the other buffer. Here’s a list of all the parameters of this function:


- target - This is the only required parameter of copy(). As we’ve seen from our previous usage, it is the buffer we want to copy to.
- targetStart - This is the index of the bytes in the target buffer where we should begin copying to. By default it’s 0, meaning it copies data starting at the beginning of the buffer.
- sourceStart - This is the index of the bytes in the source buffer where we should copy from.
- sourceEnd - This is the index of the bytes in the source buffer where we should stop copying. By default, it’s the length of the buffer.

So, to copy Nananana from wordsBuf into catchphraseBuf, our target should be catchphraseBuf like before. The targetStart would be 0 as we want Nananana to appear at the beginning of catchphraseBuf. The sourceStart should be 7 as that’s the index where Nananana begins in wordsBuf. The sourceEnd would continue to be the length of the buffers.


At the REPL prompt, copy the contents of wordsBuf like this:


```
wordsBuf.copy(catchphraseBuf, 0, 7, wordsBuf.length);


```


The REPL confirms that 8 bytes have been written. Note how wordsBuf.length is used as the value for the sourceEnd parameter. Like arrays, the length property gives us the size of the buffer.


Now let’s see the contents of catchphraseBuf:


```
catchphraseBuf.toString();


```


The REPL returns:


```
Output'Nananana Turtle!'

```


Success! We were able to modify the data of catchphraseBuf by copying the contents of wordsBuf.


You can exit the Node.js REPL if you would like to do so. Note that all the variables that were created will no longer be available when you do:


```
.exit


```


# Conclusion


In this tutorial, you learned that buffers are fixed-length allocations in memory that store binary data. You first created buffers by defining their size in memory and by initializing them with pre-existing data. You then read data from a buffer by examining their individual bytes and by using the toString() and toJSON() methods. Finally, you modified the data stored by a buffer by changing its individual bytes and by using the write() and copy() methods.


Buffers give you great insight into how binary data is manipulated by Node.js. Now that you can interact with buffers, you can observe the different ways character encoding affect how data is stored. For example, you can create buffers from string data that are not UTF-8 or ASCII encoding and observe their difference in size. You can also take a buffer with UTF-8 and use toString() to convert it to other encoding schemes.


To learn about buffers in Node.js, you can read the Node.js documentation on the Buffer object. If you’d like to continue learning Node.js, you can return to the How To Code in Node.js series, or browse programming projects and setups on our Node topic page.


