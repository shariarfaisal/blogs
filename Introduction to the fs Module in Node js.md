# Introduction to the fs Module in Node js

```Node.js```

fs, short for File System, is one of the most basic and useful modules in Node. In this article, we’ll go over some of the most important and useful methods for manipulating the file system.


# Getting Started


You’ll need to have Node.js installed first, of course. You can do that here. Luckily for us, fs is one of the ‘out-of-the-box’ modules that come with Node so it is already available to you.


```
const fs = require('fs');

```


# Synchronous vs Asynchronous


Every fs method has both a synchronous and an asynchronous version with the name for the synchronous form just ending with Sync. So an asynchronous fs.writeFile() becomes fs.writeFileSync(). Of course, Synchronous code will stop later code from running when there is an error, but they’re simpler to get the hang of so for all of the examples in this article we’ll be using their synchronous forms.


# Basic Operations


The basic CRUD operations (create, read, update, and delete) are very simple to do with only 3 main functions.


- fs.writeFileSync()
- fs.readFileSync()
- fs.unlinkSync()

## fs.writeFileSync()


fs.writeFileSync() only takes two arguments; the path to the new file’s location, which must end with the name of the new file and the data you want stored.


gator.js
```
const gators = [{
  type: 'cayman'
}];

fs.writeFileSync('./swamp/cayman.json', JSON.stringify(gators));

```


Result:


swamp/cayman.json
```
[{"type":"cayman"}]

```


Keep in mind that fs.writeFileSync() is completely rewriting the content in cayman.json so if you change type: 'cayman' to type: 'croc' and re-run the file, cayman will be replaced. This can have significant performance drawbacks when modifying large files.


## fs.readFileSync()


By default all data is returned as a ‘buffer’, a special encoded string of numbers, to fix this just pass ‘utf8’ as the second argument.


Here we’re going to return a empty array if cayman.json is empty and return the data if it exists.


gator.js
```
const gators = [{
    type: 'cayman'
}];


const getData = () => {
  let data = fs.readFileSync('./swamp/cayman.json', 'utf8');

  if (!data) return [];
  else {
    const file = JSON.parse(data);
    return file;
  }
}

const data = getData();

```


## unlinkSync()


Unlink is the simplest of the three, only requiring the path to the file or symbolic link you wish to remove.


```
fs.unlinkSync('./swamp/cayman.json');

```


# CRUD Operations On Folders


The above three methods have their own useful counterparts for manipulating directories themselves.


- fs.mkdirSync()
- fs.rmdirSync()
- fs.readdirSync()

Since the usage of fs.mkdirSync() and fs.readdirSync() are the same as their file counterparts we’ll finish up with working out the gotchas that come with fs.rmdirSync() instead.


# rimraf vs fs.rmdirSync()


The main problem with using fs.rmdirSync() to delete folders is the fact that it will only work on empty directories. If you were to try using it to remove ./swamp it will return:


```
Error: ENOTEMPTY: directory not empty, rmdir './swamp'

```


which obviously isn’t what we want. Sadly, Node does not have a native solution for this, for some reason, so we’ll have to look elsewhere.


The simplest solution I’ve found has been an npm package called rimraf, so let’s install that quickly.


```
$ npm i rimraf

```


The syntax is very similar, except instead of fs you call rimraf.sync() ( or plain rimraf() for the asynchronous version) and pass in your path as you would expect.


```
const rimraf = require('rimraf');

rimraf.sync('./swamp');

```


# Conclusion


With just these few functions you’ll be able to handle many of the common use cases with manipulating files and their data.


Any introduction to a new technology will come with obligatory referral to the documentation (it really is one of the best ways to explore something).


