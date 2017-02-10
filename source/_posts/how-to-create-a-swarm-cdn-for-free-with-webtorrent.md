---
title: How to create a Swarm (P2P) CDN for free with WebTorrent
date: 2016-03-08 14:00:00
updated: 2016-07-20 14:00:00
tags:
- webtorrent
- swarm
- cdn
- p2p
---
Have you heard about [WebTorrent](https://github.com/feross/webtorrent)? A project started by Feross Aboukhadijeh
> **WebTorrent** is a streaming torrent client for **node.js** and the **browser**. YEP, THAT’S RIGHT. THE BROWSER. It’s written completely in JavaScript — the language of the web — so the same code works in both runtimes.

Basically, it allows a file to be shared using the BitTorrent protocol, which is P2P, in the browser. Awesome right?! So let’s get started

### **Things you’ll need:**

* NPM (Comes with [NodeJS](https://nodejs.org/en/download/))

* A file you want hosted in the Swarm CDN (Any type, some file types can be rendered or even streamed in the browser)

* **CORS and HTTPS Highly Recomended**

* Motivation and/or Madness!

### Optional

* CDN… Even better right!?

## **Step 1 — Get a file**

I have selected an streamable mp4 video as the example, you can download it [**here**](https://btorrent.xyz/download#https:%2F%2Fwebseed.btorrent.xyz%2Ftimedrift.torrent) as a Web-seeded — WebTorrent or [**here**](https://webseed.btorrent.xyz/timedrift-alpine-4k-timelapse.mp4) with direct HTTPS download.

A ***webseed*** is peer that serves the pieces of the torrent through HTTP(S), allowing 24/7 seeded torrents. The pieces are requested with HTTP range requests. In the current WebTorrent implementation, each requests is up to the size of each piece.

Either way the original file is publicly available (HTTPS CORS Enabled) at the following address: [https://webseed.btorrent.xyz/**timedrift-alpine-4k-timelapse.mp4**](https://webseed.btorrent.xyz/timedrift-alpine-4k-timelapse.mp4)

You may notice that the [**βTorrent**](https://btorrent.xyz)’s URL has a link to a .torrent file ([https://btorrent.xyz/download#**https:%2F%2Fwebseed.btorrent.xyz%2Ftimedrift.torrent**](https://btorrent.xyz/download#https:%2F%2Fwebseed.btorrent.xyz%2Ftimedrift.torrent)), instead of the *mp4 *file. This is because the .torrent has the information about the pieces of the *mp4 *file and the URL of the HTTP-Accessible original file (The Webseed).

## Step 2— Upload the original file

Upload the *mp4 *video to your website’s hosting/server and make sure it’s publicly available in a URL, if it’s on a different domain, make sure it’s CORS enabled. In the example we’ll use [https://webseed.btorrent.xyz/timedrift-alpine-4k-timelapse.mp4](https://webseed.btorrent.xyz/timedrift-alpine-4k-timelapse.mp4) as the URL.

## Step 3— Create the torrent file to your file

Okay, now we need to create a .torrent file which contains the metadata and the URL where the original file is hosted. To do this, we must first install one of [WebTorrent Project’s Modules](https://github.com/feross/webtorrent#modules-1), [create-torrent](https://github.com/feross/create-torrent) [NPM Module](https://www.npmjs.com/package/create-torrent):

    npm i -g create-torrent

Once installed, we must create the .torrent. In the same folder as the original file we type the following command:

    create-torrent --urlList '[https://webseed.btorrent.xyz/timedrift-alpine-4k-timelapse.mp4](https://webseed.btorrent.xyz/timedrift-alpine-4k-timelapse.mp4')' timedrift-alpine-4k-timelapse.mp4 > timedrift.torrent

Basically, the generic command is:

    create-torrent --urlList '*<ORIGINAL FILE URL>*' *<ORIGINAL FILE NAME>* > *<TORRENT FILE NAME>*.torrent

## Step 4 — Upload the torrent file

Upload the *torrent *video to your website’s hosting/server and make sure it’s publicly available in a URL, if it’s on a different domain, make sure it’s CORS enabled. In the example we’ll use [https://webseed.btorrent.xyz/timedrift.torrent](https://webseed.btorrent.xyz/timedrift-alpine-4k-timelapse.mp4.torrent) as the URL.

## Step 5 — Setup your viewer

We now just need a static HTML & JS to render and stream this file with **WebTorrent**! The simple following code will do:

```html
<html>
  <body>
    <script src="https://cdn.jsdelivr.net/webtorrent/latest/webtorrent.min.js"></script>
    <script>
      var client = new WebTorrent()
      client.add('https://webseed.btorrent.xyz/timedrift-alpine-4k-timelapse.mp4.torrent', function (torrent) {
        // Got torrent metadata!
        console.log('Client is downloading:', torrent.infoHash)
        torrent.files[0].appendTo('body')
      })
    </script>
  </body>
</html>
```

Upload to your website’s hosting/server and you are done.

## Step 6 — Enjoy, the more viewers the faster everything is!

[![DEMO WORKING](/images/1-fHqmd0vMRXE3wKhtcTcm1Q.png)](/images/1-fHqmd0vMRXE3wKhtcTcm1Q.png)
<center>DEMO WORKING</center>

## How to do it with files

Although video is the best use case because of it size, you may want to host files and use the Swarm CDN on large files to just like [Step 1](https://btorrent.xyz/download#https:%2F%2Fwebseed.btorrent.xyz%2Ftimedrift.torrent). To do this, we must use WebTorrent’s API to create a clickable URL to the file that is available on memory once the download is complete.

In this example we’ll be downloading: [https://webseed.btorrent.xyz/juanpabloaj/other-page.md](https://webseed.btorrent.xyz/juanpabloaj/other-page.md) through WebTorrent:

```html
<html>
  <body>
    <script src="https://cdn.jsdelivr.net/webtorrent/latest/webtorrent.min.js"></script>
    <script>
      var client = new WebTorrent()
      client.add('https://webseed.btorrent.xyz/juanpabloaj/other-page.torrent', function (torrent) {
        // Got torrent metadata!
        console.log('Client is downloading:', torrent.infoHash)
        torrent.files[0].getBlobURL(function(err, url) {
          if (err) return console.log(err)
          var a = document.createElement('a')
          a.target = '_blank'
          a.download = torrent.files[0].name
          a.href = url
          a.textContent = 'Download ' + torrent.files[0].name
          document.body.appendChild(a)
        })
      })
    </script>
  </body>
</html>
```

## Finally

### Show speed, % done, etc

Finally, if you’d like to report torrent’s stats to give the user some idea of how much time/bytes are missing , check the [WebTorrent API](https://webtorrent.io/docs) and [βTorrent’s Source](https://github.com/DiegoRBaquero/BTorrent) to get an idea of how to do it.

### Instant webseed for your files

With [Peerify](https://peerify.btorrent.xyz/) you can get an instant URL to a torrent and have your original file hosted.

### Need any help?

Check WebTorrent’s [Gitter channel](https://gitter.im/feross/webtorrent) and feel free to mention me @DiegoRBaquero

## Links

* [WebTorrent Website](https://webtorrent.io/) ([GitHub](https://github.com/feross/webtorrent))

* [βTorrent Website](https://btorrent.xyz) ([GitHub](https://github.com/DiegoRBaquero/BTorrent))

I personally use [**Vultr**](http://www.vultr.com/?ref=6978436-3B) for my virtual servers. Get **$10** using my [referral link](http://www.vultr.com/?ref=6978436-3B). Thank you!

## Update 1: July 20, 2016

### HTTP 2 performance

By default, WebTorrent opens a maximum of 4 requests to a webseed simultaneously, each initial XHR on HTTP 1.1 would open each a TCP connection, and if the webseed has TLS/SSL the latency would increase for the maximum of 6 connections a browser has to a host that would be reused later for the rest of requests. Now, with the new HTTP2 protocol, a webseed can now use a single TCP connection over TLS improving the performance of a webseed. Improvement can be seen as: 83% saved in initial connections RTT latency and up to 50% faster completion of the XHR requests made to the webseed (Depends on available bandwidth).

So go ahead and activate HTTP 2 (DIRECTLY, CloudFlare wouldn’t fully work) in your hosts, not only to improve WebTorrent and webseeds but everything loaded from your servers!

A small demo using HTTP 2 can be seen [here](https://peertube.btorrent.xyz/view/b276fac02e3d2da254019ad9d13e41d6108aecab).

[![Single connection for all requests, single timeline](/images/1-QftCqcAReUVAIuBIP9-sYA.png)](/images/1-QftCqcAReUVAIuBIP9-sYA.png)*Single connection for all requests, single timeline*
