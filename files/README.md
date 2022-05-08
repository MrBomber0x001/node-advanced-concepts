# Managing files with node.js

###### Contents:

- Reading from and to Writing files

- handling streams

On this write-up we will be taking about 

- entire file [reading, writing]

- partial read or write

Reasons to Read to a file 

- load data into database: convering delimited file and loading it onto a database

- display contents of log files

the optiona avaiable to manipulating files are alot ranging from async reading to sync

to partial from full

### Reading Full File

**Reading a file asyncrounsly**

`readFile` open the file and load all the content into memory

`readFile(path, encoding, callback(err, data))`: 

if we didn't specify an encoding, the system will default to `Buffer`

```js
const {readFile} = require('fs');
readFile("./data/example.cs", "utf-8", (err, data) => {
    const vals = convertCsv(data);
    console.table(vals);
})
```

**Reading file sync**

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

In order to promisify to work, function passed to it must have the standard `callback` which follow the err first then data cb(err, data), if the order is different you'll get unexpected results

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

`open`, `read` 

`open(path, (err, fd))`

`read(fd, buffer, bufferOffset, HowMuchToRead, WhereToStartReadingOnTheFile)

bufferOffset is where to start inserting into the Buffer not the file!

if for example specifiy the bufferOffset to be 5, then the first byte read from the file will be stored on the 5 byte on the buffer

what is fd?

the os track the currently opening files on a table  assigning to each opening file an indentifier called file descriptor

on linux: integer

on windows: 32-bit handle 

fs library returns numeric descriptor for each opening file

most os have file limit

```js
const { convertCsv } = require("./csv.parse");
const { open, read } = require("fs");

open("./data/app.log", (err, fd) => {
  const buffer = Buffer.alloc(200);
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

**Reading A chunk at a time**

you need to know how big is the file, so you know how many bytes you need to read

`fs.stat` give us information about the file

```js
const { open, read, stat } = require("fs");

let totalSize = 0;
stat("./data/app.log", (err, { size }) => (totalSize = size));

open("./data/app.log", (err, fd) => {
  const buffer = Buffer.alloc(200);
  for (let i = 0; i < totalSize / buffer.length; i++) {
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

There's mainly one problem here
