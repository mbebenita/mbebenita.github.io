# MBX
Notes

# Back to Basics - June 19, 2017

This blog has gone through more blogging platforms than it has ever had blog posts.
Every few years I get the feeling that I need to write more.
That feeling is then followed by a few hours of wasted effort looking for the latest and greatest blogging platform, installing a bunch of tools, and then getting tired of it all and giving up.

This time around things are going to be different. Inspired by [HackCabin](https://hackcabin.com/post/initial-commit/#site-architecture), I'm going to keep it simple: [Markdown](https://daringfireball.net/projects/markdown/syntax), [Showdown](https://github.com/showdownjs/showdown) and [Afterdark](https://comfusion.github.io/after-dark/).

Here's the entirety of my hand rolled blogging platform:

```md
<html>

<head>
  <script src="showdown.min.js"></script>
  <link rel="stylesheet" type="text/css" href="afterdark.css" media="screen" />
</head>

<body class="hack dark main container">
  <div id="content">Processing ...</div>
  <script>
    fetch("index.md").then(function (response) {
      console.log("Rendering");
      var text = response.text().then(function (text) {
        var converter = new showdown.Converter({ tables: true });
        showdown.setFlavor('github');
        var html = converter.makeHtml(text);
        document.getElementById("content").innerHTML = html;
      });
    });
  </script>
</body>

</html>
```



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
