---
author: admin
categories:
- command line
- linux
- ubuntu
date: "2009-04-10T13:48:52Z"
slug: script-fucat-directly-to-x-clipboard
status: publish
title: 'Script-fu: cat directly to X clipboard'
wordpress_id: "44"
---

Have you ever wanted to 'cat' a file directly to the clipboard? OF
COURSE, why else would you be reading this sentence? Simply install
'xclip', which is simple also, if you are on ubuntu:

{{< highlight bash >}}
    sudo apt-get install xclip
{{< / highlight >}}
Anything that you pipe to it goes to the X11 clipboard. Once there,
you can just hit the middle mouse button to paste it where you
want, just as if you had highlighted it. So you can do things like
this:
{{< highlight bash >}}
    $ cat some.file | xclip
    $ nslookup somehost | xclip
    $ ./runsomeawesomescriptthatyouwanttheoutputintheclipboard.sh | xclip
{{< / highlight >}}


