---
layout: post
title: Compress PDF files with Quartz filters from command line
categories:
- Mac
tags:
- pdf
---

## What are Quartz filters?
Quartz is, in a nutshell, a graphic library and part of the OS X Core Graphics Framework. A Quartz filter is basically an XML file defining how Quartz should be used. Quartz filters are often used to compress PDF documents by reducing the size of the images in it – but can also be used for a variety of other image manipulations.

Here’s an article about using Quartz filters to reduce the file size of PDFs: [learn how to reduce PDF file size with a Quartz filter](http://www.tuaw.com/2007/12/12/reducing-pdf-file-size-with-a-quartz-filter/). As far as I know, you can create your own filters with the ColorSync program that comes with OS X or by writing the XML yourself. The “Reduce File Size” Quartz filter by OS X goes a bit hard on the images, making it useless for print PDFs and even a bit hard on “screen” PDFs. A guy named Jerome Colas created a few filters which give you more control over the outcome of the images. Check them out in his article “[Reduce PDF file size : free Acrobat replacement for Leopard](http://discussions.apple.com/thread.jspa?messageID=6109445&tstart=0)“.

## Using Quartz filters from command line
I wanted to use Quartz filters for exporting PDFs from an application I developed at work, preferably from the command line. It seems not that many people have wanted to do this or write about how to do it. After searching google on and off for days and trying all kinds of things, I was pointed in the right direction; you can do this with a Quartz printer application stored in /System/Library/Printers/Libraries/quartzfilter. The syntax is:

{% highlight text %}
/System/Library/Printers/Libraries/quartzfilter sourcefile filter destination
{% endhighlight %}

So converting big.pdf into small.pdf with the “Reduce File Size” filter would work like this:

{% highlight text %}
/System/Library/Printers/Libraries/quartzfilter big.pdf /System/Library/Filters/Reduce\ File\ Size.qfilter small.pdf
{% endhighlight %}

Pretty straightforward isn’t it?

## Other ways of using Quartz filters
While searching, I learned about some other ways to use Quartz filters which may be interesting for some people.

### Using Quartz filters in Python
On my mac I have the file:
*/Developer/Examples/Quartz/Python/filter-pdf.py*
I’m not sure if it came with XCode or if it’s OS X native. The script can be used to apply Quartz filters to PDF files, except well, it doesn’t work. If you have an up to date OS X, you should have at least Python 2.5 installed and it looks like the script was developed for Python ≤ 2.3 as it’s using features from the CoreGraphics library that aren’t supported in newer versions of Python. I don’t know the first thing about Python, but I figure if you’re developing with Python you might be able to fix it.

### Using Quartz filters in Automator
..is very easy. Create a new Automator project and choose the action “Apply Quartz Filter to PDF Documents” from the “PDFs” section of the actions library and use it in combination with whatever other Automator action you need.

### Using Quartz filters with AppleScript
Check this detailed article by Martin Michel over at MacScripter and try the cool droplet he made: [thoughts and examples about using the quartzfilter tool](http://macscripter.net/viewtopic.php?id=25916).

That’s it, I hope some of you can benefit from this post!
Thanks to the guys at [macosxhints](http://forums.macosxhints.com/) and [macscripter](http://macscripter.net/).

Note: I tested these things on OS X 10.5.8, I’m not sure how this will work on other versions of OS X. Please leave a comment if you find out more.
Cheers
