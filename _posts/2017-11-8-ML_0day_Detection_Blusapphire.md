---
layout: post
title: ZeroDay Detection with Machine Learning(ML) Blusapphire
category: Malware-Analysis
tags: badrabbit petya wannacry
---

Year2016 & 2017 has witnessed the rise in cyber attacks targeting various sectors like banking, industrial, etc. New variants and types (fileless/in-memory) of malware families are being surfacing each day (wannacry, Petya/NotPetya/Nyetya/Goldeneye, BadRabbit, etc) which a traditional antivirus engine couldn't detect without a signature.

With advancement in today's cybercrime, there's being advancement in detection of such potential threats, which brings me to Machine Learning (ML). According to wiki, Machine Learning (ML) is a field of computer science that gives computers the ability to learn without being explicitly programmed. In other words, computer trained to learn and identify malicious threats on its own.

Blusapphire is being integrated with Machine Learning (ML) engine that is capable of detecting any potential threats the moment they enter the network, making it easy to detect such sophisticated threats.

Last week one of our sensors has collected a file, which was flagged malicious by our Machine Learning (ML) engine. Being a zero-day, at that point in time, it has not triggered any AV flags. This post is an overview of the analysis made by Blusapphire ML engine.

----

<strong>Analyzed samples:</strong>

<table>
  <thead>
    <tr>
      <th>MD5 Hash</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>9d55d1c81605209fc2b537e74af9c91c</td>
      <td>PUP</td>
    </tr>
  </tbody>
</table>

----


<strong>Analysis:</strong>


We observed that the file was being downloaded from url ".pigcherrytoky.download"

<img src="{{ site.baseurl }}/public/0day01.jpg">


Machine learning (ML) engine has flagged the file malicious and the file is loaded with some Anti-Debug techniques, making it difficult for debugging.

<img src="{{ site.baseurl }}/public/0day02.jpg">


Being a zero-day, it has not triggered any AV flags, but the code within was matched over 176 known trojan malwares samples.

<img src="{{ site.baseurl }}/public/0day03.jpg">


Malware being multipartite, it has refused to execute in pieces.

<img src="{{ site.baseurl }}/public/0day04.jpg">


Abuse use of APIs:
{% highlight python %}
GetCommandLineA
InitializeCriticalSection
EnterCriticalSection
LeaveCriticalSection
DeleteCriticalSection
VirtualFree
VirtualAlloc
TlsDetValue
{% endhighlight %}

URL Found:
{% highlight python %}

{% endhighlight %}



Right after few hours the same PUP has being flagged malicious by multiple AV's.

<img src="{{ site.baseurl }}/public/0day05.jpg">

----
