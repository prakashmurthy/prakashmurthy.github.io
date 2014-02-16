---
layout: post
title:  "A Ajax gotcha & simple webserver with ruby"
date:   2014-02-16 21:18:41
categories: ajax, jquery, ruby, webserver
---

I have been having a very smooth sailing through [Learning jQuery book](http://www.amazon.com/Learning-jQuery-Fourth-Jonathan-Chaffer-ebook/dp/B00DMYO2KA) till being stumped by an Ajax related issue at the beginning of *Chapter 6. Sending Data with Ajax*. 

On accessing the index.html page via the browser after making the first set of modifications in the chapter, the page did not behave as expected; instead, the following error message showed up in the console:

    [Error] Failed to load resource: Origin null is not allowed by 
    Access-Control-Allow-Origin.  (a.html, line 0)
    [Error] XMLHttpRequest cannot load file:///..../06/a.html. Origin null is not 
    allowed by Access-Control-Allow-Origin. (index.html, line 0)

Googling led to [this StackOverflow answer](http://stackoverflow.com/a/8456586/429758) which made it clear that the error message was because page was being accessed from the file system instead of being accessed through a web-server. It does make sense that Ajax related functionality is accessible via a web-server, and not through the local file system. That's the Ajax gotcha I got in this context.

The solution for this is to start a webserver locally, and access the file through the **http//** urls. 

This reminded me of the following related tweet I had seen recently:
![Simple webserver with ruby command line]({{ site.url }}/images/ruby_web_server.png)

Tried it out, and VOILA! the error message is gone, and the web page behaves as expected. 

Very impressed how simple it is to start a webserver locally with a ruby command. 

Came across [this interesting blogpost](http://matteomelani.wordpress.com/2011/11/11/a-simple-web-server-is-ruby/) while trying to understand how this command line works.
