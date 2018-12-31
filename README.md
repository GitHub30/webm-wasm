# webm-wasm

webm-wasm lets you create webm videos in JavaScript via WebAssembly. The library consumes raw RGBA buffers (like `ImageData` from a `<canvas>`, for example) and turns them into a webm video with the given framerate and quality. With realtime mode you can also use webm-wasm for streaming webm videos.

The wasm module was created by [emscripten’ing][Emscripten] [libvpx], [libwebm] and [libyuv].

## Usage

webm-wasm runs in a worker by default. It works on the web and in in Node, although you need Node 11+ with the `--experimental-worker` flag.

### Quickstart
```js
// 1. Load the `webm-wasm.js` file in a worker
const worker = new Worker("webm-worker.js");
// 2. Send the path to the `.wasm` file
worker.postMessage("./webm-wasm.wasm");
// 3. Wait for the worker to be ready
await nextMessage(worker);
// 4. Send the parameters for the constructor
worker.postMessage({
  width: 512,
  height: 512
  // More constructor options below
});
// 5. Start sending frames!
while(hasNextFrame()) {
  // ArrayBuffer containing RGBA data
  const buffer = getFrame();
  worker.postMessage(buffer, [buffer]);
}
// 6. Signal end-of-stream
worker.postMessage(null);
// 7. Get the webm file as an ArrayBuffer
const webm = await nextMessage(worker);
// 8. Cleanup
worker.terminate();
```

### Constructor options

- `width` (default: `300`): Width of the video
- `height` (default: `150`): Height of the video
- `timebaseNum` (default: `1`): Numerator of the fraction for the length of a frame
- `timebaseDen` (default: `30`): Denominator of the fraction for the length of a frame
- `bitrate` (default: `200`): Bitrate in kbps
- `realtime` (default: `false`): Prioritize encoding speed over compression ratio and quality. With realtime mode turned off the worker will send a single `ArrayBuffer` containing the entire webm video file once input stream has ended. With realtime mode turned on the worker will send an `ArrayBuffer` in regular intervals.

### WebAssembly

If you just want to use the WebAssembly module directly, you can grab `webm-wasm.wasm` as well as the the [Emscripten] glue code `webm-wasm.js`.

The WebAssembly module exposes a C++ class via [embind]:

```c++
class WebmEncoder {
  public:
    // Same options as above. `cb` is a callback function that takes an ArrayBuffer.
    WebmEncoder(int timebase_num, int timebase_den, unsigned int width, unsigned int height, unsigned int bitrate, bool realtime, val cb);
    bool addRGBAFrame(std::string rgba);
    bool finalize();
    std::string lastError();
    // ...
}
```

### Experimental: TransformStreams

[Transferable Streams] are behind the “Experimental Web Platform Features” flag in Chrome Canary. The alternative `webm-transformstreamworker.js` makes use of them to expose the webm encoder. Take a look at the demos to see the usage.

## Building

Because the build process is completely [Dockerized][Docker], Docker is required for building webm-wasm.

```
$ npm install
$ npm run build
```

---
Apache 2.0

[Docker]: https://www.docker.com/
[Transferable Streams]: https://www.chromestatus.com/features/5298733486964736
[embind]: https://developers.google.com/web/updates/2018/08/embind
[Emscripten]: https://kripken.github.io/emscripten-site/
[libvpx]: https://github.com/webmproject/libvpx
[libwebm]: https://github.com/webmproject/libwebm
[libyuv]: https://chromium.googlesource.com/libyuv/libyuv/