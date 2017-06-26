# Speech or Music? - June 19, 2017

[Opus](https://opus-codec.org/) is a highly versatile audio codec.
In fact, it's comprised of two codecs: Skype’s SILK codec which works well for speech and Xiph.Org’s CELT codec which is best suited for music. Opus picks which codec to use by analyzing the input signal. If the signal sounds like music it uses CELT, if it sounds like speech then it uses SILK, if there's no signal (silence, or noise) then it just uses whatever it was using previously. Opus make this choice every 20 ms (or the typical length of an audio frame.)

This sounds like a fun, and fairly contained problem. A great opportunity to learn about signal processing and machine learning. Opus currently ships with a simple neural network that does this classification (a single hidden layer of 15 neurons.) Can I do better? Can you? I invite you to try.

[More ...](?page=speech_or_music.md)


# AV1 Bitstream Analyzer - June 19, 2017

Here at Mozilla, we’ve been hard at work on the new AV1 video codec. AV1 aims to improve coding efficiency by 25% over HEVC (h.265) and VP9, and is developed by the [Alliance of Open Media](http://aomedia.org/) of which Mozilla is a part of.

[More ...](?page=aomanalyzer.md)

# Back to Basics - June 19, 2017

This blog has gone through more blogging platforms than it has ever had blog posts.
Every few years I get the feeling that I need to write more.
That feeling is then followed by a few hours of wasted effort looking for the latest and greatest blogging platform, installing a bunch of tools, and then getting tired of it all and giving up.

[More ...](?page=basics.md)



<!--
# HEADER

ABC

## A.1
ABC

### A.1.1
ABC

#### A.1.1.1
ABC

##### A.1.1.1.1
ABC

[Get Showdown!](https://google.com)

| Tables        | Are           | Cool  |
| ------------- |:-------------:| -----:|
| **col 3 is**  | right-aligned | $1600 |
| col 2 is      | *centered*    |   $12 |
| zebra stripes | ~~are neat~~  |    $1 |-->


