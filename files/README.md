# Managing files on node

## Table of Contents

- Reading from and Writing to files
  - Partial read/write
  - Full read/write
  - Different methods used (Sync, Async)
  - Applying OS level permissions
- watching for files
- Handling streams:
  - reading/writing
  - handling back pressure
- Moving and Deleting (linking and unlinking)
- Renaming

### Reasons to Read to a file

You'll face tasks such as

- loading data into database: convering delimited file and loading it onto a database

- displaying contents of log files

the option avaiable to manipulating files are alot ranging from async reading to sync and from  partial to full

### Reading Entire File

**Reading a file asyncrounsly**

`readFile` **open** the file and **load** all the content into memory

`readFile(path, encoding, callback(err, data))`.
the first parameter of the callback is `error` because node follow the first-error callback philosophy

printing the error is very important to avoid losing the real cause of incoming errors
if we didn't specify an encoding, the system will default to `Buffer` (buffer of bytes)

```js
const {readFile} = require('fs');
readFile("./data/example.cs", "utf-8", (err, data) => {
  if(err) {
    console.log(`There was a problem with the file ${err}`)
    return;
  }
    const vals = convertCsv(data);
    console.table(vals);
})
```

**Reading file sync**

wrap the `readFileSync` with `try/catch` block to catch errors
the `sync` methods do not use an `error` parameter instead they `throw` errors which cause system to crash

```js
const {readFileSync} = require('fs');
try{
    const data = readFileSync("./data/example.cs", "utf-8");
    console.table(convertCsv(data));
} catch(error){
    throw new Error(`An Error occured: ${error}`)
}
```

**Reading file using promisify**

In order to promisify to work, the actual function passed to it must have the standard `callback` which follow the err first then data cb(err, data), if the order is different you'll get unexpected results.

```js
const { promisify } = require("util");
const readFile = promisify(fs.readFile);
readFile("./data/pulitzer-circulation-data.csv", "utf-8")
  .then((data) => {
    console.table(convertCsv(data));
  })
  .catch((err) => {
    throw new Error(`Error happend`);
  });

const readAsync = async () => {
    const data = await readFile("./data/example.cs", "utf-8");
    console.table(convertCsv(data));
}
readAsync();
```

### Reading File Partially

sometimes you need to view the last 10 requests of a log file containing 1000000 of  requests

reading the entire file will cause the file to be loaded into the memory which consumes more space and we don't need this type of situation

so Reading it partially will help us decrease the load and if we want to make processes on the the data it would be possible

`open`, `read`

`open(path, (err, fd))`

`read(fd, buffer, bufferOffset, HowMuchToRead, WhereToStartReadingOnTheFile)`

- `bufferOffset` is where to start inserting into the Buffer not the file!, so it's not the offset of the file
- `buffer` is the place where you will start reading into

if for example specifiy the bufferOffset to be 5, then the first byte read from the file will be stored on the 5 byte on the buffer

what is fd?

the os track the currently opening files on a table  assigning to each opening file an indentifier called file descriptor

on linux: integer

on windows: 32-bit handle

fs library returns numeric descriptor for each opening file, node uses this file descriptor to lookup the file from the operating systems

most os have file limit, so it's important to close the file after doing the operation

```js
const { convertCsv } = require("./csv.parse");
const { open, read } = require("fs");

open("./data/app.log", (err, fd) => {
  const buffer = Buffer.alloc(200); // allocate a buffer of 200 bytes long to be read at a time
  // count parameter hold the number of actual bytes read
  // buff is the data read
  read(fd, buffer, 0, buffer.length, 0, (err, count, buff) => {
    if (err) {
      console.log(err);
      return;
    }
    console.log(count);
    console.table(convertCsv(buff.toString()));
  });
});
```

There's one problem with this approach, we're only enable to read just `200 bytes` of the file, the solution is to read it chunk by chunk

**Reading A chunk at a time**

you need to know how big is the file, so you know how many bytes you need to read

`fs.stat` give us information about the file

```js
const { open, read, stat } = require("fs");

let totalSize = 0;
stat("./data/app.log", (err, { size }) => (totalSize = size));

open("./data/app.log", (err, fd) => {
  const buffer = Buffer.alloc(200);
  for (let i = 0; i <= totalSize / buffer.length; i++) {
    read(
      fd,
      buffer,
      0,
      buffer.length,
      i * buffer.length,
      (err, count, buff) => {
        console.table(buff.toString());
      }
    );
  }
});
```

There's mainly one problem here:
read is an asynchrounous method and we are executing it inside a for loop, so no gurantee of the order that function will return, so each time, it will return different result

the solution is to read `sync`
**Reading chunk at a time**

```js
const {openSync, readSync} = require("fs")
const fd = openSync("./data/app.log")
let count = 0;

do {
 const buffer = Buffer.alloc(200); // the buffer created inside the do statement
  count = fs.readSync(fd, buffer, 0, buffer.length, null); // this is a signicant change
  console.log(buffer.toString())
} while (count > 0)
```

making the position `null` will make node able to track where in the file and pick up there the next time trying to read.

```js
for (let i = 0; i < 50000; i++){
  const fd = fs.openSync("./data/app.log")
  console.log(fd) //where does it crash
} // crash 
```

```js
for (let i = 0; i < 50000; i++){
  const fd = fs.openSync("./data/app.log")
  console.log(fd) 
  fs.closeSync(fd)// there's no difference between close and closeSync
}
```

what about `readFile`, should we close it ?
well it depends!
If you passed a file path to `readFile`, then it closes it automatically, if you passed a `fd` then you must close it

> Any time you've a file descriptor, you are responsible for closing the file

### Writing Entire file

`writeFile(path, data, options, cb(err))` default the flag option to `w`

```js
const {writeFile}=require("fs")

writeFile("./data/app.log", "welcome", (err) => {
  err ? console.log(err) : console.log("file saved!")
});

writeFile("./data/app.log", "welcome", {flag: "a"}, (err) => {
  err ? console.log(err) : console.log("file saved!)
})
```

`appendFile` default the flag to `a`

```js
const {appendFile} = require("fs")
appendFile("./data/app.log", "welcome", (err) => {
  err ? console.log(err) : console.log("file saved!")
})
```

#### playing with options

**main flags**:
`r` read mode
`w` write mode
`a` append mode
**Extended flags**:
can be added to the main flags
`x` (exclusive flag) which can be added to any flag, throw an error in case if the file already exist
`+` open in multiple mode, in case of `w+` it will create the file if it doesn't exist, but `r+` will throw an error
`s` synchrouns, it will not convert `open` to `openSync` for example, it has to deal this the file I/O
**read option**
There are 3 allowed combination of reading options
`r`, `r+`, `rs`
**writing option**
`w`, `wx`, `w+`, `wx+`
**append option**
`a`, `ax`, `a+`, `ax+`, `as`, `as+`

```js
const {writeFile} = require("fs")
writeFile("./data/app.log", "welcome", {flag: "wx"}, (err) => {
  err ? console.log(err): console.log("file saved!")
})
```

We can add os level permissions on the file while reading or writing
| fs.constatns | Ocatal | Description |
| -- | -- | -- |
| S_IRUSR | 0o400 | Read by owner |
| S_IWUSER | 0o200 | Write by Owner |

```js
const { constants, writeFile } = require("fs")

writeFile("./text.js", "console.log(`welcome`)", {
    mode: constants.S_IWUSR | constants.S_IRUSR // === 0o600
}, (err) => {
    err ? console.log(err) : console.log("file saved!")
})

{ mode: 0o600 } // create read and write permission for the user and no permission for any user or any users in a group
// you can look for the Permission constants table  
```

We can specifiy also an encoding
`{encoding: "base64"}`

:small_airplane: before writing to the file, all of the content gets loaded into memory first

`path` module allows for cross-platform usage

1. `.resolve` get the absolute path from relative one

2. `.normalize` Normalizes any path by removing instances of `.` , turning double slashes into single slashes and removing a directory when `..` is found.

3. `.join` join strings to a directory

```js
const path = require("path");
console.log(path.resolve("index.js")); //root/bin/desktop/etc
console.log(path.normalize("./app//src//util/.."));
// app/src
console.log(path.join("/app", "src", "util", "..", "/index.js"));

// prints  /app/src/index.js
```

```js
import {promises as fsPromises} from 'fs';
const writeData = async () => {
    try{
    const openedFile = await fsPromises.open("writefile.txt", "a+");
    let newFile = await fsPromises.writeFile("writeFile.txt", "hello world");
    await openedFile.write("welcome");
    } catch(err){
        console.error(err);
    }
}
writeData();


const readFile = async () => {
    try{
        const buff = new Buffer.alloc(26); //space for 26 characters
        let openedFile = fsPromises.open('readFile.txt', "a+");
        await (await openedFile).read(buff, 0, 26);
        let readEntireFile = await fsPromises.readFile("writeFile.txt", "utf-8");
        console.log(readFile);
        console.log(buff);
    } catch(err){
        console.log(err);
    }

}
```

1. .write()

   1. does not overwrite content

   2. file must be open first

      ```js
      write(data, options)
      ```

2. .writeFile()

   1. overwrites existing file

      ```js
      writeFile(path, data, options)
      ```

***reading***

1. .read()

   1. read entire file

   2. read part of a file

   3. file must be opened first

   4. requires a buffer to store read data

      ```js
      .read(buffer, options)
      ```

2. .readFile()

   1. read entire file

   2. more popular choice

      ```js
      .readFile(path, options)
      ```

**moving and renaming**

- renaming and moving are the same

- .rename()

- change path argument

- use .mkdir() if the directroy doesn't exist

`.rename(original_path, new_path)`

**Deleting files and directories**

- `.unlink()` method to remove file

- `.rmdir()` remove dir, will fail if the dir is not empty

- you can use 3rd path module `rimraf` to delete directories with files

`unlink(path), rmdir(path)`

### Real world Sceniro

Suppose we need to get an `index.js` file which contains all of our nested other files exported into that index.js file?
like this

```js
module.exports.synchrounsRead = require('synchronous.read.js').read
module.exports.synchrounsWrite = require('synchronous.write.js').write
```

Let's get started, our procedure:

1. Open file `index.js`
2. Get list of files on each directory
3. Iterate over list of files
4. add the `module.exports ...` statement to the name
5. write to the file
6. close the file

`watch(dir or file, event listener(eventType, fileName))`

```js
const { closeSync, openSync, writeSync, readdirSync, watch } = require("fs")

watch("./read", () => {
  const indexFd = openSync("./index.js", 'w'); // open for writing

  const files = readdirSync("./read");

  files.map(f => {
      const name = f.replace("js", "")
      console.log(`Adding a file: ${f}`);
      writeSync(indexFd, `module.exports.${camelCase(name)} = require('./ read/${name}).read;\n`)
  })
closeSync(indexFd)

})
    // `module.exports.fileName = require('file.name').read`

// npm install camelCase
```

the process of watch will run again if you changed/add the name of the file or made any change to it's content

### Reading and Writing Streams

#### Reading

With streams you don't read the stream with a function, instead, you create a stream and recieve data via an event

with each read, the buffer contains `64kB`, because this is the default size of a stream, however, you do have the ability to change that default size which is called `highWaterMark`
`highWaterMark` is the max number of bytes the stream will read at one time

```js
const { createReadStream } = require("fs");


const stream = createReadStream("./text.js", { highWaterMark: 1, encoding: "utf-8" });

let count = 0;
stream.on("data", (data) => {
    count++
    console.log(count);
    console.log(data);
})

```

Sometimes the stream of data can move faster than your code can handle, how are going to handle those situations?
`.resume()` and `.pause()` come into play

```js
const { createReadStream } = require("fs");

const stream = createReadStream("./text.js", { highWaterMark: 1, encoding: "utf-8" });

stream.on("data", (data) => {
    stream.pause();
    data = data.toUpperCase()
    console.log(data);

    setTimeout(() => { // we are simulating that our coding something before resuming
        stream.resume();
    }, 2000)
})
```

#### Writing

Writing is no different from reading

```js
const { createReadStream, createWriteStream } = require("fs");

const stream = createReadStream("./text.js", { highWaterMark: 1, encoding: "utf-8" });
const writer = createWriteStream("./anotherText.js")

let iteration = 0;
stream.on("data", (data) => {
    stream.pause();
    console.log(iteration++);

    writer.write(data);
    setTimeout(() => { // we are simulating that our coding something before resuming
        stream.resume();
    }, 1000)
})
```

🤔 What happens when the output stream is slower than the input stream?
As long as the data incoming from input stream can be handled from output stream, even if the output stream is faster than input streams, still no problem!

`Back Pressure`:
A backup of data, caused by streams being unable to process data before the next batch arrives

The `stream.log` is about 1.8 GB

```js
const stream = createReadStream("./data/stream.log", {
  encoding: "utf8"
})

const writer = createWriteStream("./data/output.log");

let iteration = 0;

stream.on("data", data => {
  console.log(++iteration);

  writeDate(data);
})

const writeDatea = data => {
  setTimeout(() => {
    writer.write(data);
  }, 60000)
}
```

This code however produces an error causing the whole process to stop writing, because the heap memory of js is loaded to full, and can't process more data.
to overcome this issue, we need just one line

```js
const stream = createReadStream("./data/stream.log", {
  encoding: "utf8"
})

const writer = createWriteStream("./data/output.log");

stream.pipe(writer)
```

as with `.pipe` it will not load all of the content into memory and also will not throw a memory exception error
it will take care of the `back pressure` problem and `highWaterMark`

## Resources

- Udacity Advanced full stack web nanodegree.
- Managin files on nodejs, plurasight
