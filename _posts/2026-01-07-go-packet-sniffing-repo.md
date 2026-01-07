---
layout: post
title: New packet-sniffer tool
subtitle: Extend the implementation to support more packet inspector modules in this project
tags: [shit, linux, sniffing]
author: VuNguyen
---

---

I had this idea yesterday because I got a task about research error responses of Redis those are sent back to clients/applications several days ago. I didn't spend much time to research this task, because it was similar with MySQL tracing tool I done before, I discussed about that in <a style="color: blue" href="./2026-01-02-happy-new-year-2026.md">this post</a>

But, maybe there are many problems when I want to implement more than one tool like this, it has similar codebase, directory structure, even function name. Copy project from one to another and rename it, logic is only changed in a function, blah blah ... All of these aren't always a good idea. At current time, I only have MySQL, Redis, but in the future it may have more (for example: Kafka, Postgres, MongoDB). Not to mention that if I need to add a function on all of these tool, I need to update in all of project, this obivously is nightmare.

**I very very don't like to do the repeatable work**

So, I created a new project with a new codebase (this project was renamed from old project, which has a name **mysql-error-echo**). Project link: <a href="https://github.com/vuvietnguyenit/packet-sniffing" style="color: blue">https://github.com/vuvietnguyenit/packet-sniffing</a>

I only take around 15min to add new Redis sniffer module, export to Prometheus metrics, inherit functions are existed before, that is very convenient. Moreover, I won't scary about add new functions on these sperate projects, because all project is one project. But to achieve this, you need a sufficiently good codebase.

_Ah, in the <a style="color: blue" href="./2026-01-02-happy-new-year-2026.md">previous post</a>, I mentioned about performance when use sniffing technique. I built a testing idea like this:_

![testing flow](/assets/images/Mermaid%20Chart%20-%20Create%20complex,%20visual%20diagrams%20with%20text.-2026-01-07-064011.png)

I used two machine to test latency about Redis client -> server, send request from machine 1 and get response from machine 2. I put BCC (base on eBPF) latency measurement tool <a href="https://github.com/iovisor/bcc/blob/master/tools/tcpconnlat_example.txt" style="color: blue">tcpconnlat</a> on machine 1 to measure latency of "spammer" PID when enable/disable sniffer tool on machine 2 that I described in image above.

And it give me a result: "**Latency didn't change when I enabled/disabled `sniffer` tool on machine 2**". If you can see "Something is wrong" in my testing idea, let's me know it. Thanks.
