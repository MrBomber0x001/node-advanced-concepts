# What happens when we require() a module?

## There are 5 steps in this process:

- Resolving & Loading.
- Wrapping.
- Execution.
- Returning Exports.
- Caching.

### **Step 1: Resolving & Loading**:

> Before explaining this step, You should know that there are 3 types of modules:

- **Core modules** such as `(fs, http, etc..)`
- **Your modules** such as (`./utils/someUtil.js`)
- **3rd party modules** such as (`express`)

- **Process**:
  - Start with core modules.
  - If the module is begin with a relative path such as `./` **or** `../`, It will try to load **Your modules**.
  - If there's no file, It will try to see if it's a folder and load the `index.js` from it.
  - else, It will try to go `node_modules` and try to find it there, might be a `3rd party module`.
  - otherwise, It will raise an `error`.

### **Step 2: Wrapping**:

- Our module gets wrapped into an **Immediately invoked function expression** Aka `IIFE` which expose objects to use like `exports`, `require`, `module`, `__filename` and `__dirname`

- **This will achieve 2 things**:
  - Get access to those exposing objects.
  - Keeps our Top-level code variables private, because They will be scoped only to the current module.
- ```js
  (function (exports, require, module, __filename, __dirname) {
    // Module's Implementation Code
  })();
  ```

### **Step 3: Execution**:

- Here, the execution of our module starts to happen.

### **Step 4: Returning Exports**:

- Here, Any exports are done by either way (`module.exprots` or `exports.someFunc`) will be returned to be required again in other modules

### **Step 4: Caching**:

- The module is cached after the first time is loaded.
- The code inside the module is executed only once.
- In subsequent calls, the result will be retrieved from the cache.
