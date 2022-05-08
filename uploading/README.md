## Uploading Files using Node

On the end of this article we are gonna make a good project "Image resize api" 

We have two or more main ways 

1. using native upload 
2. using middleware to give us more flexibility

On this articles I will talk about the two ways on much more details 

### Using Native Approach

- sending files from a web browser
- how to receive uploaded files
- sending files back to the client

## Introduction

Data types used in RESTful Communication

- Primitives
- More complex objects (are serialized in json format)

for example: 

- Strings
- Numbers
- Boolean
- Objects
- Arrays
- Files (Binary)

In this article we are going to focus on files

1. Uploading single file at a time:
   1.1 Classic HTML forms
   1.2 Styling File input tags
   1.3 XMLHttpRequests 
   
       1.3.1 Sync
   
       1.3.2 async
   
       1.3.3 tracking progress
   
   1.4 Fetch API

2. uploading multiple files
   
   2.1 HTML Forms
   
   2.2 XMLHttpRequest
   
   2.3 Fetch API

3. Managing Files on the Server
   
   1. Receiving files
   2. sending files
   3. downloading files

### Uploading single file

**1.1 Classic HTML Forms**

 `multipart/form-date` tell the browser that incoming data are big and require special handling

```html
<form method="post" action="..." enctype="multipart/form-data">
    <input id="textdoc" type="file" name="kitten_pic">
    <button type="submit">Submit</button>
</form>
```

By default your form will submit with method "GET"

when we submit the form, we are leaving the current page to the action url!

special attributes and properties `accept` is used for filtering

```html
<input type="file" accept="plain/text, .pdf, application/msword">
<script>
    const files = document.getElementById('textdoc').files //FileList, this allow use to get actual information about the files being submitted by the client
</script>
```

##### Demo

```html
<form action="/api/single-upload" enctype="multipart/form-data" method="post">
    <input type="file" name="myfile">
    <button type="submit"> upload </button>
</form>
```

```js
// On the server
const express = require("express");
const app = express();
const path = require("path").win32;
const fileUpload = require("express-fileupload");
app.use(express.static("public"));
app.use(express.text());
app.use(fileUpload());
app.use(express.raw({ type: "image/*", limit: "5mb" }));

app.pst("/api/single-upload", (req, res) => {
  const contentType = req.header("content-type");
  if (contentType.includes("text/plain")) {
    res.set("Content-Type", "text/plain");
    res.set(req.body);
  } else if (contentType.includes("multipart/form-data")) {
    const f = req.files.myfile;
    res.set("Content-Type", "text/html");
    f.mv("./uploads/" + f.name);
    res.send(`
      <table>
          <tr><td>Name</td><td>${f.name}</td></tr>
          <tr><td>Size</td><td>${f.size}</td></tr>
          <tr><td>MIME type</td><td>${f.mimetype}</td></tr>
      </table>
  `);
  } else {
    res.set("Content-Type", contentType);
    res.send(req.body);
  }
});
app.post("/api/multi-upload", (req, res) => {
  const contentType = req.header("content-type");
  if (contentType.includes("text/plain")) {
  }
});
app.listen(3000, () => {
  console.log("server is running on port 3000");
});
```

**1.2 Styling File Input Tags**

There are some limitation to using `Classic html input file`: 

- Limites style options

- Styles are not consistent between browsers

However, we can have a hack around those issues, we are gonna hide the input tag and upload using `JS`

```html
<input type="file" style="display: none">
```

##### Demo

```html
  <form
      action="/api/single-upload"
      method="post"
      enctype="multipart/form-data"
    >
      <input type="file" name="myfile" id="myfile" style="display: none" />
      <button type="button" id="uploadButton">Select File</button>
      <div>Selected File: <span id="filename"></span></div>
      <button type="submit">upload</button>



  </form>
  <script>
  console.log("working!");

const fileTag = document.getElementById("myfile");
const filenameElem = document.getElementById("filename");
const uploadButton = document.getElementById("uploadButton");

uploadButton.addEventListener("click", () => {
  fileTag.click();
});
fileTag.addEventListener("change", () => {
  filenameElem.innerText = fileTag.files[0].name;
});
</script>
```

**1.3 XMLHttpRequests**

XHR is used to asynchrounsly send data without leaving (refershing the current page)

so we are actually getting the data from the server on the current page.

However, XHR can be used synchrounsly: blocking the user from doing anything till we get response back from the client

**1.3.1 send sync**

##### Demo

```html
    <form
      action="/api/single-upload"
      method="post"
      enctype="multipart/form-data"
    >
      <input type="file" name="myfile" id="myfile" />
      <button type="submit">upload</button>
    </form>
    <button id="submitXHR">submit XHR</button>
```

```js
const submitXHR = document.getElementById("submitXHR");
const form = document.querySelector("form");

submitXHR.addEventListener("click", () => {
  const xhr = new XMLHttpRequest();
  xhr.open("post", "/api/single-upload", fa
lse);// false here mean (send sync)
  xhr.send(new FormData(form));
});
```

notice that we didn't leave the current page!

**1.3.2 send async**

when using async XHR, before doing anything, we have to set up event listeners

```js
xhr.responseType = 'blob'; //expecting to receive blob data [images, file, so on;]
xhr.addEventListener('onreadystatechange', () => {
    if(xhr.readyState === xhr.DONE){
        // success
}
})
```

##### Demo

```html
    <form
      action="/api/single-upload"
      method="post"
      enctype="multipart/form-data"
    >
      <input type="file" name="myfile" id="myfile" />
      <button type="submit">upload</button>
    </form>
    <button id="submitXHR">submit XHR</button>
    <img />
```

```js
const submitXHR = document.getElementById("submitXHR");
const input = document.querySelector("input");
const img = document.querySelector("img");
submitXHR.addEventListener("click", () => {
  const xhr = new XMLHttpRequest();
  xhr.open("post", "/api/single-upload");
  xhr.responseType = "blob";
  xhr.addEventListener("readystatechange", () => {
    if (xhr.readyState === xhr.DONE /* 4 */) {
      img.src = URL.createObjectURL(xhr.response);
    }
  });
  xhr.send(input.files[0]);
});
```

`createObjectURL` is used to constructor a url from a blob or metadata

and the src of the image will be on that format

`blob:http://localhost:3000/89903baa-5e8e-4643-bc41-be6645241dae`

**1.3.3 Tracking progress upload**

basically the most important thing about xhr is `tracking upload`

it's useful to give user some indication about the state of the file being uploaded if the file is merely long enough!

on xhr, we use event called `progress` and we get obhect back passing to the callback

this event going to be called each time the upload is running!

##### Demo

we are just going to add additional event as we talked eariler 

```js
xhr.addEventListner("progress", (e) => {
     console.log(
      `Percent Complete: ${Math.round((e.loaded / e.total) * 100)} %`
    );
})
```

open up your console, and notice the percentage of the file being uploaded, this all depends on your network speed!

**1.4 Fetch API**

fetch returns a `Promise<Response>`

##### Demo

```js
const submitFetch = document.getElementById("submitFetch");
const input = document.querySelector("input");
const img = document.querySelector("img");
submitFetch.addEventListener("click", () => {
  fetch("/api/single-upload", {
    method: "post",
    body: input.files[0],
  })
    .then((res) => res.blob())
    .then((data) => (img.src = URL.createObjectURL(data)));
});
```

### Uploading Multiple files

**2.1 HTML Forms**

Here we add simple additional attribute to the `input` element called `multiple` to indicate we are sending multiple files, and on the client js we can references those files by one of the two methods 

```html
<form action="/api/multiple-upload" enctype="multipart/form-data" method="post">
    <input type="file" name="myfile" multiple>
    <button type="submit"> upload </button>
</form>
```

```js
const files = document.querySelect("input");
const f = files[0] // or 
let fi = files.item(0);
```

Our upload api for managing multiple files 

```js
app.post("/api/multi-upload", (req, res) => {
  const files = req.files.myfiles;
  let response = "<table>";
  for (const f of files) {
    f.mv("./uploads/" + f.name);
    response += `
    <tr>
        <td>Name: ${f.name}</td>
        <td>Size: ${f.size}</td>
        <td>MIME type: ${f.mimetype}</td>
    </tr>
  `;
  }
  response += "</table>";
  res.send(response);
});
```

**2.2 XMLHttpRequest**

Nothing will change from the previous example of sending single file

```js
const form = document.querySelector("form");

document.getElementById("submitXHR").addEventListener("click", () => {

const xhr = new XMLHttpRequest();

xhr.open("POST", "/api/multi-upload", false);

xhr.send(new FormData(form));

});
```

##### Demo of tracking

```js
const input = document.getElementById("myfiles");

const submitXHR = document.getElementById("submitXHR");
submitXHR.addEventListener("click", () => {
  for (const f of input.files) {
    console.log(f);
    const xhr = new XMLHttpRequest();
    xhr.responseType = "blob";
    xhr.addEventListener("readystatechange", () => {
      if (xhr.readyState === xhr.DONE) {
        const img = document.createElement("img");
        img.src = URL.createObjectURL(xhr.response);
        document.body.appendChild(img);
      }
    });
    xhr.addEventListener("progress", (e) => {
      console.log(
        `Filename: ${f.name} Percent Complete: ${Math.round(
          (e.loaded / e.total) * 100
        )} %`
      );
    });
    xhr.open("post", "/api/single-upload");
    xhr.send(f);
  }
});
```

The most important note here: 

we are sending a single file per request, so we used the `single-upload` api route for handling this

**2.3 Fetch API**

```js
const input = document.getElementById("myfiles");

const submitFetch = document.getElementById("submitFetch");
submitFetch.addEventListener("click", () => {
  for (const f of input.files) {
    fetch("/api/single-upload", {
      body: f,
      method: "POST",
    })
      .then((res) => res.blob())
      .then((data) => {
        const img = document.createElement("img");
        img.src = URL.createObjectURL(data);
        document.body.appendChild(img);
      });
  }
});
```

### Managing Files on the server

**Recieving file from the client**

we have two buit-in methods and one 3rd party library for handling 

- express.text() handle plain text files

- express.raw() handle binary files and make the request as a buffer 

- expresss-fileupload => package for handling multipart/form-data and give the req additional property called `files` which then we can access the filenames from it:
  
  - value is file for single upload
  
  - value is object containing key-value pairs of filenames and content for multi upload
  
  - this package give us a method `mv` to move this file or files to my file system

```js
app.use(fileUpload())
app.post('/single-upload', (req, res) => {
const f = req.files.myfile;
f.mv('./uploads' + f.name); 
})
```

notice here there is no security check here, you have to move the files to an area where you do a virus scan regularly or sort of security scanning from time to time.

also don't move those files into the public directory, because it will be accessible to all users which can spread malicious software on that directory

handling multiple files is pretty much the same!

**Sending files to the client**

we have 3 main options

- res.sendFile() -> uses absolute path

- res.download() -> uses relative path

- res.attatchment() -> relative path

we've now under public directoy an image of anything called 'example.png';

```js
app.get('/api/download', (_req, res) => {
    res.download('./public/example.png', 'yourNewName.png');
})
app.get('/api/sendFile', (_req, res) => {
    res.sendFile(path.join(__dirname, "public/example.png"));
})
app.get('/api/attatchment', (_req, res) => {
    res.attatchment('./public/example.png')
})
```

**Download File on the client**

1. Anchor Tag

2. Fetch API

```html
<a href="example.png">Download example</a>
```

```js
document.getElementById("downloadImage").addEventListener("click", () => {
    fetch('example.png').then(res => res.blob()).then((data) => {
    const img = document.createElement("img");
    img.src = URL.createObjectURL(data);
    document.body.appendChild(img);
})
})
```

### Using middleware

There are plenty of middleware available on npm, but I've chosen 
Multer as my main uploading middleware 

## File Upload

I will talk about raw file upload

then Multer

Multer

read more about Multer for better information

multer accept only form-data/multipart

can be configured multiple times, if for example i need to accept on a route pdf files only, and on another route i need to accept images and so on

**Validation**

```js
const upload = multer({
    dest: 'uploads',
    limites: {
        fileSize: 10000
    },
    fileFilter(req, file, cb){
           cb(new Error("File must be a pdf"));
           cb(undefined, true)
            cb(undefined, flase)
    }
})
```
