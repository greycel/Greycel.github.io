---
layout: post
title: BadRabbit Closer Look with BluSapphire
category: Malware-Analysis
---

Since the outbreak of Petya/NotPetya which was surfaced in the month of June, again last week a new ransomware attack "aka BadRabbit" is making the headlines effecting machines in Ukraine, Russia, Turkey and Bulgaria.

----

<strong>Initial Attack Vector:</strong>

Unlike Petya/NotPetya that use SMB (Eternal Blue) as the initial vector, this variant uses drive-by-download type of attack to deliver the malware (BadRabbit) that spreads via malicious websites.

#### BadRabbit utilizes:
1.   Diskcryptor to encrypt the files with selected extensions
2.   SCmanager, schtasks and rundll32.exe to invoke other components
3.   For lateral movement, it scans the local networks for SMB shares and spread via SMB
4.   Mimikatz for credential harvesting on compromised machine

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
      <td>fbbdc39af1139aebba4da004475e8839</td>
      <td>Adobe_Flash_Update – Dropper</td>
    </tr>
  </tbody>
  <tbody>
    <tr>
      <td>1d724f95c61f1055f0d02c2154bbccd3</td>
      <td>infpub.dat – Main DLL</td>
    </tr>
    <tr>
      <td>b4e6d97dafd9224ed9a547d52c26ce02</td>
      <td>cscc.dat – Driver for Encryption</td>
    </tr>
    <tr>
      <td>b14d8faf7f0cbcfad051cefe5f39645f</td>
      <td>dispci.exe – DiskCryptor Client</td>
    </tr>
  </tbody>
</table>

----


<strong>Behavioral analysis:</strong>

Once downloaded, the executable dropper pretending to an Adobe Flash Update convincing the victim to install it

<img src="{{ site.baseurl }}/public/bad00.jpg">



Upon execution it drops the main module DLL "infpub.dat" in "C:\Windows" directory that is further initiated by rundll.exe with arguments.
{% highlight python %}
C:\Windows\system32\rundll32.exe C:\Windows\infpub.dat,#1 15
{% endhighlight %}
<img src="{{ site.baseurl }}/public/bad01.jpg">



Executes the command “schtasks /Delete /F /TN rhaegal” to delete any existing tasks with name “rhaegal”.

<img src="{{ site.baseurl }}/public/bad02.jpg">


During the execution of main DLL "infpub.dat" other components  (cscc.dat, dispci.exe) responsible for encrypting are being dropped.

<img src="{{ site.baseurl }}/public/bad03.jpg">


To launch the newly dropped components of diskcryptor “dispci.exe” on the startup, a new task is scheduled with name “rhaegal”.

<img src="{{ site.baseurl }}/public/bad04.jpg">


New service named “cscc” is created for DiskCryptor Driver “cscc.dat”.
{% highlight python %}
ServiceName=cscc,DisplayName=Windows Client Side Caching DDriver, BinaryPathName=cscc.dat
{% endhighlight %}
<img src="{{ site.baseurl }}/public/bad05.jpg">


Schedules a task named “drogon” to forcefully reboot the machine at 04:46hrs, it appears that a reboot is required to install the DiskCryptor drivers.
<img src="{{ site.baseurl }}/public/bad06.jpg">
<img src="{{ site.baseurl }}/public/bad07.jpg">

BadRabbit encrypts only selected file extension as below and display ransom note.
{% highlight python %}
3ds, 7z, accdb, ai, asm, asp, aspx, avhd, back, bak, bmp, brw, c, cab, cc, cer, cfg, conf, cpp, crt, cs, ctl, cxx, dbf, der, dib, disk, djvu, doc, docx, dwg, eml, fdb, gz, h, hdd, hpp, hxx, iso, java, jfif, jpe, jpeg, jpg, js, kdbx, key, mail, mdb, msg, nrg, odc, odf, odg, odi, odm, odp, ods, odt, ora, ost, ova, ovf, p12, p7b, p7c, pdf, pem, pfx, php, pmf, png, ppt, pptx, ps1, pst, pvi, py, pyc, pyw, qcow, qcow2, rar, rb, rtf, scm, sln, sql, tar, tib, tif, tiff, vb, vbox, vbs, vcb, vdi, vfd, vhd, vhdx, vmc, vmdk, vmsd, vmtm, vmx, vsdx, vsv, work, xls, xlsx, xml, xvd, zip
{% endhighlight %}

<img src="{{ site.baseurl }}/public/bad000.jpg">

Abuse use of APIs:
{% highlight python %}
CloseHandle
CreateFileW
CreateProcessW
ExitProcess
GetCommandLineW
GetCurrentProcess
GetFileSize
GetModuleFileNameW
GetModuleHandleW
GetSystemDirectoryW
HeapAlloc
ReadFile
TerminateProcess
UnhandledExceptionFilter
WriteFile
{% endhighlight %}

URL Found:
{% highlight python %}
http://rb.symcb.com/rb.crl0W
http://s.symcb.com/universal-root.crl0
http://ocsp.verisign.com0
https://www.verisign.com/rpa
https://www.verisign.com/rpa0
http://rb.symcb.com/rb.crt0
http://ts-crl.ws.symantec.com/sha256-tss-ca.crl0
https://d.symcb.com/cps0%
http://s.symcd.com06
http://crl.verisign.com/pca3-g5.crl04
http://ts-ocsp.ws.symantec.com0;
https://d.symcb.com/rpa0@
https://d.symcb.com/rpa0
https://www.verisign.com/cps0
https://d.symcb.com/rpa06
http://crl.thawte.com/ThawteTimestampingCA.crl0
http://s.symcd.com0
http://ocsp.thawte.com0
https://d.symcb.com/rpa0.
http://rb.symcd.com0&
http://ts-crl.ws.symantec.com/tss-ca-g2.crl0(
http://sf.symcb.com/sf.crt0
http://ts-aia.ws.symantec.com/sha256-tss-ca.cer0(
http://logo.verisign.com/vslogo.gif04
http://sf.symcd.com0&
http://ts-aia.ws.symantec.com/tss-ca-g2.cer0<
http://sf.symcb.com/sf.crl0W
http://ts-ocsp.ws.symantec.com07
{% endhighlight %}

----


<strong>Lateral Movement:</strong>

To perform credential harvesting, it creates and loads mimikatz to a file with extension “.tmp” (xxxx.tmp) in “C:\Windows\” and initiates a new process from the temp file “495E.tmp” with pipe.

<img src="{{ site.baseurl }}/public/bad08.jpg">
<img src="{{ site.baseurl }}/public/bad09.jpg">

Noticed that the malware scans the local network for ports 139, 445 and spreads via SMB shares with credentials harvested using mimikatz.

<img src="{{ site.baseurl }}/public/bad10.jpg">
<img src="{{ site.baseurl }}/public/bad11.jpg">
