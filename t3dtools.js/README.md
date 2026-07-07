# T3DTOOLS.js

## COPYRIGHT

The code in this repository comes from many different places:

- https://github.com/ahom
- https://github.com/RequestTimeout408
- The `squish` library is licensed under the MIT license by Simon Brown (si@sjbrown.co.uk)

---

## Overview

This project is a port of **t3dgw2tools** compiled with **Emscripten**.

`t3dtools.js` provides utilities to process raw binary data extracted from Guild Wars 2 `.dat` archives before parsing.

It can:

- Decompress compressed byte arrays
- Decode/unpack textures
- Handle GW2-specific binary formats

The output is an Emscripten WebAssembly module that can run in:

- Node.js
- Web browsers
- Web Workers

Since Emscripten 2.x, the build generates a JavaScript loader together with its corresponding `.wasm` binary.

---

## Usage (Node.js)

Example:

```javascript
const fs = require("fs");

require("./t3dtools.js").then((t3dtools) => {
    const compressed = fs.readFileSync("chunk.packed");

    const data = t3dtools.inflate(compressed);

    console.log(data);
});
````

---

# Build Requirements

Required:

* CMake 3.20+
* Ninja (recommended)
* Emscripten SDK

Tested with:

* Windows 10/11
* Latest Emscripten SDK
* CMake + Ninja generator

---

# Installing Emscripten

Example:

```cmd
git clone https://github.com/emscripten-core/emsdk.git C:\dev\emsdk

cd C:\dev\emsdk

emsdk install latest
emsdk activate latest
```

Initialize the environment:

```cmd
emsdk_env.bat
```

Verify:

```cmd
emcc -v
```

---

# Building

## Windows

Open an Emscripten-enabled terminal:

```cmd
cd C:\dev\emsdk
emsdk_env.bat
```

Go to the project directory:

```cmd
cd t3dtools.js
```

Configure with Emscripten:

```cmd
emcmake cmake -G Ninja .
```

Build:

```cmd
cmake --build .
```

or:

```cmd
ninja
```

---

## Clean rebuild

If changing compiler, Emscripten version, or CMake generator, remove the CMake cache:

```cmd
del CMakeCache.txt
rmdir /s /q CMakeFiles
```

Then configure again:

```cmd
emcmake cmake -G Ninja .
cmake --build .
```

---

# Build Output

After a successful build, the output contains:

```
t3dtools.js
t3dtools.wasm

t3dworker.js
t3dworker.wasm
```

The JavaScript file is the Emscripten module loader.
The `.wasm` file contains the compiled C/C++ code.

---

# t3dtools.js vs t3dworker.js

## t3dtools.js

Normal module.

Runs directly in:

* Node.js
* Browser main thread

Suitable for:

* Command-line tools
* Server-side extraction
* Simple browser usage

## t3dworker.js

Web Worker version.

Runs heavy operations in a worker thread.

Useful for:

* Browser applications
* Avoiding main-thread blocking
* Large texture decompression jobs

---

# Notes

## Emscripten version compatibility

This project was originally created for older Emscripten versions.

Modern Emscripten versions may require small CMake adjustments:

* Use `emcmake cmake`
* Use Ninja instead of Makefiles
* Avoid Unix-style shell quoting in linker flags

---

# Known Issues / Future Improvements

* Replace exception handling with explicit error checking because Emscripten exception support is limited.
* Use `SharedArrayBuffer` instead of copying data between threads.
* Improve error reporting during decompression.
* Add more automated tests for texture formats.

```