---
title: "Compiling dcraw to WebAssembly"
tags: [webassembly, javascript, wasm]
categories: blog
published: true
comments: false
---

At EyeEm we handle about every digital photo format you can imagine. And with the variety of digital cameras out there, some of our web-based tools must accept [RAW photo formats](https://en.wikipedia.org/wiki/Raw_image_format). Sometimes a JPG just won't make the cut.

RAW photo file sizes can be huge. Depending on which camera you're using, they can vary from 10MB to well over 100MB.

That's of course impractical for web use. I want to take these huge RAW files and compress them for web usage so we can have thumbnail previews. And I want to do this without a round-trip to another server.

## Goals

1. Process RAW images in the browser. We **don't** want to do this server-side.
2. Leverage WebAssembly for speed and memory safety.

## Emscripten

> [Emscripten](https://emscripten.org/index.html) is a toolchain for compiling to asm.js and WebAssembly, built using LLVM, that lets you run C and C++ on the web at near-native speed without plugins.

How magical does that sound? It also sounds a little bit like things might get complicated. However though, I found that was **not** the case. In fact I was genuinely in awe of how much Emscripten can do with minimal configuration.

## dcraw

David Coffin wrote a brilliant [open source library](https://www.dechifro.org/dcraw/) written in C called `dcraw` which supports decoding RAW files from 731 different digital cameras. It's been around since the '90s and seems pretty battle-tested. There's even a [port](https://github.com/zfedoran/dcraw.js) of `dcraw`to Node.js by Zelemir Fedoran. And while this port **can** run as plain Javascript in the browser through some neat `MEMFS` tricks, it does not leverage WASM for the heavy lifting.

Looking at Fedoran's [build process](https://github.com/zfedoran/dcraw.js/blob/master/makefile) gave me a great starting point for building `dcraw` with Emscripten's `emcc` tool. I want to leverage the speed of WASM too, so my end-goal is to compile `dcraw` so we can call it directly in the browser with all the speed that the WASM binary gives us.

## Extracting embedded JPGs from a RAW file

Did you know that **most RAW files include an embedded JPG thumbnail**? These may be used for low-penalty previews in a camera's display screen. Well no wonder RAW files are so big. The size and resolution of these JPGs can vary a bit depending on the camera model, so you might want to run some checks on what you extract. I've seen some extracted preview JPGs that are only 400px wide, which doesn't meet our size requirements for thumbnail resolution.

So how do we extract these?

`dcraw` includes an `-e` (extract?) flag, which will return an extracted thumbnail from a RAW file, if it has one. If you were running it as a command-line tool, you would do so like:

```bash
$ dcraw -e <filename>
```

## First pass

Let's first try the simplest `emcc` build targeting WASM.

```bash
$ emcc dcraw.c -s WASM=1

dcraw.c:77:10: fatal error: 'jasper/jasper.h' file not found
#include <jasper/jasper.h>      /* Decode Red camera movies */
         ^~~~~~~~~~~~~~~~~
1 error generated.
shared:ERROR: '/usr/local/opt/emscripten/libexec/llvm/bin/clang -target asmjs-unknown-emscripten -D__EMSCRIPTEN_major__=1 -D__EMSCRIPTEN_minor__=38 -D__EMSCRIPTEN_tiny__=44 -D_LIBCPP_ABI_VERSION=2 -Werror=implicit-function-declaration -Xclang -nostdsysteminc -Xclang -isystem/usr/local/Cellar/emscripten/1.38.44/libexec/system/include/libcxx -Xclang -isystem/usr/local/Cellar/emscripten/1.38.44/libexec/system/lib/libcxxabi/include -Xclang -isystem/usr/local/Cellar/emscripten/1.38.44/libexec/system/include/compat -Xclang -isystem/usr/local/Cellar/emscripten/1.38.44/libexec/system/include -Xclang -isystem/usr/local/Cellar/emscripten/1.38.44/libexec/system/include/libc -Xclang -isystem/usr/local/Cellar/emscripten/1.38.44/libexec/system/lib/libc/musl/arch/emscripten -Xclang -isystem/usr/local/Cellar/emscripten/1.38.44/libexec/system/local/include -DEMSCRIPTEN dcraw/dcraw.c -Xclang -disable-O0-optnone -Xclang -isystem/usr/local/Cellar/emscripten/1.38.44/libexec/system/include/SDL -c -o /var/folders/4k/bv2brsrx6cb5r4b20320_gp00000gp/T/emscripten_temp_HVI35t/dcraw_0.o -emit-llvm' failed (1)
```

Ah, looks like there are some extra dependencies in `dcraw`. Luckily its documentation says we have the option to compile without them!

So I passed this extra flag like so.

```bash
$ emcc dcraw.c -lm -DNODEPS -s WASM=1

...
shared:ERROR: BINARYEN_ROOT must be set up in .emscripten
```

Another error, which may be specific to my OS X environment. It's a strange message, since after installing `emscripten` with Homebrew, it specifically instructed me to comment-out the `BINARYEN_ROOT` line in my `.emscripten` file. Luckily, I found [this Github comment](https://github.com/Homebrew/homebrew-core/issues/47869#issuecomment-566700418) which had a working fix.

So, I installed `binaryen` manually.

```bash
$ brew install binaryen
```

Then opened `~/.emscripten` and set

```bash
BINARYEN_ROOT = '/usr/local/opt/binaryen'
```

Now once again, I tried

```bash
$ emcc dcraw.c -lm -DNODEPS -s WASM=1
```

🎉 And it worked! A few non-fatal warnings were generated, but so were two files:

-   `a.out.js`
-   `a.out.wasm`

We have our WASM file, and a Javascript "glue" file which lets us interface with it.

Those file names are pretty meaningless though, so let's pass another flag to specify the output names.

```bash
$ emcc dcraw.c -lm -DNODEPS -s WASM=1 -o dcraw.js
```

Which yields

-   `dcraw.js`
-   `dcraw.wasm`

### So how do we call our WebAssembly module?

First things first, let's just call that Javascript file in a basic page, and see if anything happens in the dev tools console.

```html
<!doctype html>
<script async src="dcraw.js"></script>
</html>
```

Sure enough! It prints the full `man` page for `dcraw` as if we were using it in a shell environment with no arguments!

<img src="https://user-images.githubusercontent.com/547148/72154445-eeb01800-33b0-11ea-912b-97c3727a4b83.png" alt="dcraw browser console output" />

The Javascript file generated from Emscripten has automatically imported our WASM binary in the background, and run `dcraw` in the WASM sandbox, printing the contents of `stdout`. Pretty cool.

But we don't want to immediately run `dcraw`, which is what's happening by default. So let's disable that by adding `INVOKE_RUN=0` and a named runtime method when we do our build.

```bash
$ emcc dcraw.c -lm -DNODEPS -s WASM=1 -s INVOKE_RUN=0 -s EXTRA_EXPORTED_RUNTIME_METHODS='["callMain"]' -o dcraw.js
```

By doing this, we can now invoke `dcraw` later, from a `callMain` function.

```html
<!doctype html>
<script>
  var Module = {
    onRuntimeInitialized: () => {
      // Set a 2 second delay to test!
      setTimeout(() => Module.callMain(), 2000);
    }
  };
</script>

<script async src="dcraw.js"></script>
</html>
```

And we see the same output, but delayed by two seconds. The `Module.callMain()` function is now what executes `dcraw`.

It's worth noting that **only** `main()` from `dcraw.c` is called here by default, which is good enough for us right now.

> By default, Emscripten-generated code always just calls the `main()` function, and other functions are eliminated as dead code. <sup>[MDN](https://developer.mozilla.org/en-US/docs/WebAssembly/C_to_wasm)</sup>

So we are calling `dcraw`, but it's pretty useless without any files or flags. Let's add those.

`Module.callMain()` can receive one array of arguments. The last one must always be a **reference** to a file buffer stored as a `Uint8Array`. All precending ones can be flags like `-e`, `-c`, etc. It will take some work to get us there, but it would look like

```javascript
Module.callMain(["-e", "raw_file_buffer"]);
```

So how do we store a RAW file buffer as a `Uint8Array` in MEMFS, and then pass a reference to it?

```html
<!doctype html>
<script>
  var Module = {
    onRuntimeInitialized: async () => {
      const image = await fetch("IMG_4248.CR2").then(r =>
        r.arrayBuffer()
      );

      // Cast ArrayBuffer to Uint8Array
      const clampedBuffer = new Uint8Array(image);

      // Create a workspace in MEMFS
      FS.mkdir("/workspace");
      FS.chdir("/workspace");
      FS.writeFile("raw_file_buffer", clampedBuffer);

      // The buffer is stored in FS. Pass a reference to its name.
      Module.callMain(["-e", "raw_file_buffer"]);

      console.log(FS.readdir("/workspace"))
      /* Will print
         [ ".", "..", "raw_file_buffer", "raw_file_buffer.thumb.jpg" ]
      */
    }
  };
</script>

<script async src="dcraw.js"></script>
</html>
```

Great! We can already see the name of the extracted thumbnail in the FS workspace we created! All that's left is to read this new extracted thumbnail from the workspace, and clean it up.

```html
<!doctype html>
<script>
  var Module = {
    onRuntimeInitialized: async () => {
      const image = await fetch("IMG_4248.CR2").then(r =>
        r.arrayBuffer()
      );

      // Cast ArrayBuffer to Uint8Array
      const clampedBuffer = new Uint8Array(image);

      // Create a workspace in MEMFS
      FS.mkdir("/workspace");
      FS.chdir("/workspace");
      FS.writeFile("raw_file_buffer", clampedBuffer);

      // The buffer is stored in FS. Pass a reference to its name.
      Module.callMain(["-e", "raw_file_buffer"]);

      const extracted = FS.readFile("raw_file_buffer.thumb.jpg", {encoding: "binary"});

      // Cleanup
      if (extracted && clampedBuffer) {
        ["raw_file_buffer", "raw_file_buffer.thumb.jpg"].forEach(item => {
          FS.unlink(item);
        })
      }
      FS.chdir("/");
      FS.rmdir("/workspace");

      // Now use `extracted` to create an image Blob to be used in the browser
      const blob = new Blob([extracted], {type: "image/jpeg"});
      const imageUrl = URL.createObjectURL(blob);
      const img = document.createElement("img");
      img.src = imageUrl;
      img.width = 600;
      document.body.appendChild(img);
    }
  };
</script>

<script async src="dcraw.js"></script>
</html>
```

And there we have it. Our most minimal implementation of `dcraw` compiled to WASM, extracting a RAW image thumbnail and rendering it directly in the browser.

Hopefully the steps outlined are straightforward enough to go forth and compile your own C/C++ libraries to WASM. The piece that I got hung-up on the most was using the browser's in-memory `FS`. It wasn't obvious to me that WASM was able to write directly in there, until I started logging everything in the workspace I created.

If you compile anything of your own and want to share with the world, or got stuck on any part of this, just ping me on Twitter [@ultrasandwich](https://twitter.com/ultrasandwich).

## Example demo

If you would like to see the full code example from this article (plus a little extra), you can find the Github repo at [https://github.com/eschaefer/wasm-dcraw](https://github.com/eschaefer/wasm-dcraw).

[A running demo is also here](https://eschaefer.github.io/wasm-dcraw).

## Further reading

-   [dcraw home page](https://www.dechifro.org/dcraw)
-   [Compiling dcraw to node](https://github.com/zfedoran/dcraw.js/blob/master/makefile)
-   [Compiling an Existing C Module to WebAssembly](https://developer.mozilla.org/en-US/docs/WebAssembly/existing_C_to_wasm) - _MDN_
-   [Compiling a New C/C++ Module to WebAssembly](https://developer.mozilla.org/en-US/docs/WebAssembly/C_to_wasm) - _MDN_
-   [Building projects with Emscripten](https://emscripten.org/docs/compiling/Building-Projects.html)
-   [Squoosh](https://github.com/GoogleChromeLabs/squoosh) - Open source, encode/decode many image formats in the browser using WASM
