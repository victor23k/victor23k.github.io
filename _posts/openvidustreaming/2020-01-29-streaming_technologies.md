---
layout: post
category: ops
title: Streaming technologies
author: Víctor Fernández
---

***


In this last decade the increase of live streaming platforms and use cases has caused a race to achieve the **lowest latency posible**. Acceptable numbers can vary depending on the use case. For real-time communication, it is generally considered that the experience starts to degrade above 200ms of latency, beyond this limit conversations start to become more [1]. Live sports are also very sensitive. This comparison shows diferent live streaming protocols and latency [2]:

![graphic](/assets/streamingprotocolslatency.png)

HTTP based protocols have higher latency than WebRTC, but CDNs rely on HTTP so those protocols have an already existing infrastructure and thus have better scalability as we will see.

## HTTP based live streaming

HTTP based live streaming protocols like HLS and MPEG-DASH divide a video stream into small chunks which reside on an HTTP server. In this way the individual video chunks can be downloaded by a video player via TCP.

<img src="/assets/httpstreaming.png" height="500"/> [4]

This allows the video to traverse firewalls easily, where it can be delivered as it is watched. This minimizes caching and enables viewers to seek to a different position just by retrieving the corresponding video chunks. Moreover, it is scalable because it allows Content Delivery Networks (CDN)
to leverage the capacity of their HTTP networks to deliver streaming content instead of relying upon smaller networks of dedicated streaming servers [3]. Most used HTTP based protocols [Apple's HTTP Live Streaming][HLS] (HLS) and international standard Dynamic Adaptive Streaming over HTTP (MPEG-DASH) rely on this technique and suffer the same problem. 

### Scalability

1. Content is recorded, encoded and uploaded to a media server over an  TP-based connection.
2. The media server decodes the inbound stream and re-encodes it at varying bit-rates in “small” file chunks.
3. These chunks are distributed out to content distribution servers [1 thru “n”]. The number and geograph-ic location of these servers can vary, but ideally should be located as closely as possible to the content subscriber to minimize latency.
4. Clients start downloading these chunks and buffering them for playback. This is where the most latency is introduced, anywhere from 10-30 seconds.
5. Client-side code analyzes the network traffic determine which bitrate to use for the next downloaded chunk. [4]

### Availability

Making an HTTP stream highly available means uploading the content to at least two different media servers and ensuring that each content distribution node runs with at least two servers with the same content available on each. Since everything is HTTP-based, load balancing on the download side is relatively straight-forward - just add an HTTP load balancer.


## RTC streaming

WebRTC streaming offers very low buffering time (ms). The total latency is upload latency plus server processing tume plus download latency plus jitter buffer duration, and this number is usually between 500ms and 300ms even with not ideal Internet connectivity.

<img src="/assets/webrtclatency.png" height="500"/> [4]

### Scalability

Scaling a WebRTC stream requires a large network of live-repeating servers in order to handle the load. According to WebRTC expert Tsahi Levent-Levi, whenever you connect one browser to another with a direct stream, you have to create and utilize a peer connection. But there is a limit to the number of concurrent peer connections a browser will allow; on Chrome, the limit is 500 connections, but Levant-Levi recommends no more than 50. [5]

### Availability

Making an WebRTC highly available implies adding even more forwarding servers.

## HTTP based vs RTC based 

This table summarizes cons and pros for these lines of implementation.


|      | HTTP                               | WebRTC |
|------|----------------------------------- |--------|
| Pros |High quality, scalability and availability  |Low latency |
| Cons |High latency                        |Top realistic quality is 1080p30fps, great cost to make scalable and highly available|

***
### References
[1] : [Akamai Whitepaper](https://www.akamai.com/us/en/multimedia/documents/white-paper/low-latency-streaming-cmaf-whitepaper.pdf)

[2] : [MUX Blog](https://mux.com/blog/the-low-latency-live-streaming-landscape-in-2019/)

[3] : [Red5Pro Whitepaper](https://www.red5pro.com/white-paper-gateway/)

[4] : [Frozen Mountain Live Broadcasting Whitepaper](https://www.frozenmountain.com/media/1301/frozenmountain-irtc-live-broadcasting.pdf)

[5] : [Wowza What is WebRTC](https://www.wowza.com/blog/what-is-webrtc)

[HLS]: https://developer.apple.com/streaming/