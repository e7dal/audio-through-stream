Through stream for audio processing.

* Compatible with node-streams (pcm streams).
* Can be piped right to [speaker](https://npmjs.org/package/speaker).
* Shares _AudioBuffer_ between connected instances instead of copying _Buffer_, which is 0-overhead.
* Uses zero-watermarks to avoid output delays.
* Provides an easy way to control the flow pressure, e. g. to bind processing to the real time, debug chunks, outsource processing to shaders/webworkers/audio-workers, etc.
* Provides debugging facilities.
* Provides simple audio data metrics.
* Can be used as a _Readable_, _Transform_ or _Writable_ stream.
* Provides WAA interface to output stream to web audio destination.
* WIP: .plan method to schedule events by audio-time.


## Usage

[![npm install audio-through](https://nodei.co/npm/audio-through.png?mini=true)](https://npmjs.org/package/audio-through/)

```js
var Through = require('audio-through');
var util = require('audio-buffer-utils');
var Speaker = require('speaker');

var through = new Through(function (buffer, done?) {
    //returning null ends stream
    if (this.time > 3) return null;

    var volume = 0.2;

    util.fill(buffer, function (sample) {
      return sample * volume;
    });

    cb(null, buffer);
  }, {
    //automatically act as generator if piped anywhere
    gnerator: true,

    //pcm options, in case if connected to raw output stream
    sampleRate: 44100,
    channels: 2,
    samplesPerFrame:
  }
);


//Throw error, not breaking the pipe
through.error(error|string);

//Log buffer-related info
through.log(string);

//Set true to display stream logs/errors in console. `false` by default.
Through.log = true;
```

Through constructor takes `process` function and `options` arguments.

Processor function receives `buffer` and optional `done` callback, and is expected to modify buffer or return a new one.

`buffer` is an instance of _AudioBuffer_, used as input-output. It is expected to be modified in-place to avoid "memory churn" in real-time streams. Still, if a new buffer is returned then it will be used instead of the `buffer`.

Callback argument can be omitted, in that case processor does not hold stream and releases data instantly, like a sink. (The pattern reminisce mocha tests.). If callback argument is present, stream will wait till callback’s invocation.
Callback receives two arguments — `done(error, data)`, default node callbacks convention.

Processor function has varialbes at command:

* `this.count` — number of processed samples.
* `this.frame` — number of processed frames (chunks).
* `this.time` — time of the beginning of current chunk, in seconds.
* `this.sampleRate`, `this.channels`, `this.samplesPerFrame` — params of audio-stream, see [pcm-util](http://npmjs.org/package/pcm-util) for complete list.

### Inherit

```js
function MyProcessor (opts) {
  Through.call(this, null, opts);
};

inherit(MyProcessor, Through);


MyProcessor.prototype.process = function (buffer, done) {
  //modifying buffer code
};
```


## Related

> [audio-generator](https://github.com/audio-lab/audio-generator) — audio signal generator stream.<br/>
> [audio-speaker](https://github.com/audio-lab/audio-speaker) — output audio stream in browser/node.<br/>
> [audio-shader](https://github.com/audio-lab/audio-shader) — shader-based audio processing stream.<br/>
> [audio-buffer](https://github.com/audio-lab/buffer) — audio data holder.<br/>
> [audio-buffer-utils](https://npmjs.org/package/audio-buffer-utils) — set of utils for audio buffers processing.<br/>
> [pcm-util](https://npmjs.org/package/pcm-util) — utils for low-level pcm stream tasks.<br/>