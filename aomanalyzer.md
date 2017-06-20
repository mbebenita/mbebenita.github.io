# AV1 Bitstream Analyzer - Updated June 19, 2017

Here at Mozilla, we’ve been hard at work on the new AV1 video codec. AV1 aims to improve coding efficiency by 25% over HEVC (h.265) and VP9, and is developed by the [Alliance of Open Media](http://aomedia.org/) of which Mozilla is a part of.

AV1 is a derivative of VP9 and now includes a large number additional coding tools and experiments imported from [Daala](https://xiph.org/daala/), [Thor](https://github.com/cisco/thor/commits/master) and VP10. These experiments interact with each other in intricate and unexpected ways, and must be carefully tested on a wide variety of content. This can take a long time to complete. A single video frame can sometimes take more than an hour to encode and as part of our routine testing, we encode 30 clips, each containing 60 frames. The encoding process is massively parallel and runs on a large number of AWS instances, but even with all that hardware, it can take hours or even days to run a single test job.

This computational cost makes it inconvenient for developers to encode and analyze videos locally. But it also uncovers an interesting question. If all of our testing and build infrastructure runs in the cloud, why not run all of our analysis tools in the cloud (or browser) as well?

Our first attempt at this is the bitstream analyzer. The analyzer decodes AV1 bitstreams and displays a variety of details about a bitstream. This information can help codec engineers more easily identify and fix bugs. The input to the analyzer is usually small (an encoded bitstream), but the output is very large. For instance, a single 1080p video frame produces 4MB of raw image data and a large amount of analyzer metadata. This is usually not a problem if the analyzer runs locally, but if the analyzer runs remotely on a server, then bandwidth and latency becomes a big concern.

The ideal solution is to run the analyzer directly in the browser and thus eliminate the need to download analyzer output.
To do this, we need to port the analyzer and the codec to JavaScript (luckily we have a tool that does that for us, Emscripten thanks to Alon Zakai and many others) and run it inside the browser.

The analyzer is made up of two components: decoder.js which is the [Emscripten](http://kripken.github.io/emscripten-site) compiled version of the codec and an HTML based UI front-end.

To analyze a video, all we need to do is specify a video file (in *.ivf format.) and an appropriate decoder.js file to decode it.

```
analyzer.html?decoder=decoder.js&file=a.ivf&file=b.ivf
```

In the link above, analyzer.html loads the decoder and decodes 2 bit streams a.ivf and b.ivf with it. Alternatively, multiple decoders can be used to analyze videos:

```
analyzer.html?decoder=aDec.js&file=a.ivf&decoder=bDec.js&file=b.ivf
```

The beauty of this approach is that you can easily share links to decoded videos, and all of it runs in the browser, no need to maintain any server infrastructure. Decoder JavaScript files are automatically generated for each revision of the codec that is submitted to [AreWeCompressedYet.com](https://arewecompressedyet.com/) (AWCY), so they are publicly accessible.


[Voila, click me!](https://arewecompressedyet.com/analyzer/?maxFrames=4&decoder=https://arewecompressedyet.com/runs/ll4-cdef-var1g@2017-04-14T15:55:28.732Z/js/decoder.js&decoderName=ll4-cdef-var1g%402017-04-14T15%3A55%3A28.732Z&file=https://arewecompressedyet.com/runs/ll4-cdef-var1g@2017-04-14T15:55:28.732Z/objective-1-fast/Netflix_Crosswalk_1920x1080_60fps_8bit_420_60f.y4m-63.ivf)

Or, of course you could:

1. Follow the directions here: http://aomedia.org/contributor-guide/ (which I recommend you do if you want to help out.)
2. Check out a particular revision of the codec that is compatible with the encoded video you want to analyze.
3. Build and run the hypothetical local analyzer.
$. If you want to share some of analyze results, take a screenshot and share it, or ask a colleague to repeat steps 1
through 3. Which of course, they are less likely to do, because it takes more than a click.

> I work at a browser company, so I may be biased, but I think this is the web at its finest.

Emscripten is often used to port games or C/C++ libraries to the web, and this is really no different, but it’s a slightly different use case that I haven’t seen before. We use Emscripten to make our continuous integration build artifacts runnable and shareable, how cool is that? Beat that convenience, native!

## Playing with the Analyzer

[More ...](https://hackernoon.com/av1-bitstream-analyzer-d25f1c27072b)