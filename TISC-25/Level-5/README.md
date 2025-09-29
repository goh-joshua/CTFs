# Level-5 - SYNTRA

DESCRIPTION

<p align="center"><img width="489" height="289" alt="image" src="https://github.com/user-attachments/assets/460c04d3-069d-45bf-822d-976ac7bb14ec" /></p>


It looks harmless enough. A jukebox, streaming random music tracks from some unknown source. You press play, it plays music. The buttons work, the dials turn, and there is a faint LED glowing just enough to remind you it is still watching.

But this is not just some forgotten novelty.

Rumors suggest that devices like this were never meant for entertainment. They were built for something else entirely. Devices made specially to broadcast messages covertly, carefully designed to blend in as a regular electronic gadget. Those in the know call it the SYNTRA, Syndicate Transceiver Array.

We seized this unit during an operation targeting individuals linked to Spectre, the same group responsible for the chaos we thought had been buried. However, there seems to have been some countermeasures built into this unit to prevent further analysis by our team. Whether this is a leftover relic from earlier operations or something that is still relevant, no one can say for certain. It might be nothing, or it might be exactly what we need to finally get closer to the kingpin.

Your task is to investigate the SYNTRA and see if you can find any leads.

IMPORTANT: ALL THE LETTERS IN THE FLAG ARE UPPERCASE

attached files
syntra-server

---

Okay, back to actual challenges.

We are given a server file and a link to a website.

I played around with the website and had no idea what I was doing. So I found a free copy of IDA PRO 9.2 online and tried to use it for the first time with my friend CHAT.

After trying to understand what the helly was happening, I found the main_main function and kept clicking until I found the three functions which look important.

- main_computeMetricsBaseline
- main_determineAudioResource
- main_evaluateMetricsQuality

And in main_determineAudioResource my neuron activated as I saw:

```code
if ( (unsigned __int8)main_evaluateMetricsQuality(a1) )
  {
    result._r0 = "assets/flag.mp3HalfClosedLocalapplication/pdfapplication/oggfont/collectionapplication/zipnegative updateaccept-encodingaccept-languagex-forwarded-forhttp2xconnect=1trailers_pseudobad_path_methodAccept-EncodingPartial ContentRequest TimeoutLength RequiredNot ImplementedGateway Timeoutunexpected typebad trailer keynegative offsetnot a directorycopy_file_range476837158203125[^a-zA-Z0-9/-]+X-Forwarded-For' in new path 'invalid argSize<invalid Value>allocmRInternalGC (fractional)write heap dumpasyncpreemptoffforce gc (idle)sync.Mutex.Lockruntime.Goschedmalloc deadlockruntime error: scan missed a gmisaligned maskruntime: min = runtime: inUse=runtime: max = requested skip=bad panic stackrecovery failedmorestack on g0stopm holding pstartm: m has ppreempt SPWRITEmissing mcache?ms: gomaxprocs=randinit missed]\n\tmorebuf={pc:: no frame (sp=runtime: frame ts set in timertraceback stuckunexpected kind: cannot parse ,M3.2.0,M11.1.0x509keypairleafrecord overflowbad certificatePKCS1WithSHA256PKCS1WithSHA384PKCS1WithSHA512ClientAuthType(unknown versionclient finishedserver finishedAccept-Languagemissing addressunknown network/etc/mime.types()<>@,;:\\\"/[]?=Hanifi_RohingyaPsalter_Pahlavino such processadvertise errornetwork is downno medium foundkey has expiredjstmpllitinterp(?i)<(/?)script is unavailablereflectlite.Settarinsecurepathx509usepolicieszipinsecurepath";
    result._r1 = 15; # I was clicking and pressing buttons until I somehow made the variable above like that.
  }
```

But anyways, even I could understand this. We just want main_evaluateMetricsQuality(a1) to be non-zero!

So I went into that function:

```code
// main.evaluateMetricsQuality
__int64 __golang main_evaluateMetricsQuality(__int64 a1)
{
  unsigned __int128 v1; // kr00_16
  __int64 v2; // rdx
  __int64 v3; // rsi
  __int64 v4; // rdx
  __int64 v5; // rcx
  main_ActionRecord *v6; // rdi
  __int64 v7; // r8
  _1_main_ActionRecord *p__1_main_ActionRecord; // rax
  __int64 v10; // rbx
  __int64 v11; // rdx
  __int64 v12; // rcx
  __int64 v13; // rbx
  __int64 v14; // rdx
  __int64 i; // rsi
  int v16; // r10d
  int *v17; // rax
  __int64 v18; // rbx
  __int64 v19; // [rsp+0h] [rbp-50h]
  __int64 v20; // [rsp+8h] [rbp-48h]
  __int64 v21; // [rsp+10h] [rbp-40h]
  __int64 v22; // [rsp+20h] [rbp-30h]
  __int64 v23; // [rsp+20h] [rbp-30h]
  main_ActionRecord *v24; // [rsp+28h] [rbp-28h]
  __int64 v25; // [rsp+38h] [rbp-18h]
  main_ActionRecord *v26; // [rsp+40h] [rbp-10h]

  v1 = (unsigned __int128)main_computeMetricsBaseline();
  v17 = (int *)v1;
  v18 = *((_QWORD *)&v1 + 1);
  v2 = *(_QWORD *)(a1 + 24);
  if ( *((__int64 *)&v1 + 1) > v2 )
    return 0;
  v3 = *(_QWORD *)(a1 + 16);
  v25 = v3;
  v4 = v2 - 1;
  v5 = 0;
  v6 = 0;
  v7 = 0;
  while ( v4 >= 0 && v7 < v18 + 5 )
  {
    if ( *(_DWORD *)(v3 + 12 * v4) != 4 )
    {
      v21 = v4;
      v20 = v5;
      v22 = 3 * v4;
      v24 = v6;
      v19 = v7;
      p__1_main_ActionRecord = (_1_main_ActionRecord *)runtime_newobject(&RTYPE__1_main_ActionRecord);
      v10 = v20;
      v11 = *(_QWORD *)(v25 + 4 * v22 + 4);
      *(_DWORD *)p__1_main_ActionRecord = *(_DWORD *)(v25 + 4 * v22);
      *(_QWORD *)&(*p__1_main_ActionRecord)[0].Value = v11;
      if ( (unsigned __int64)(v20 + 1) > 1 )
      {
        p__1_main_ActionRecord = (_1_main_ActionRecord *)runtime_growslice(v20, &RTYPE_main_ActionRecord, v11, 1);
        v12 = v13;
        v10 = v20;
      }
      else
      {
        v12 = 1;
      }
      v23 = v12;
      v26 = (main_ActionRecord *)p__1_main_ActionRecord;
      runtime_memmove(&(*p__1_main_ActionRecord)[1], v24, 12 * v10);
      v7 = v19 + 1;
      v17 = (int *)v1;
      v4 = v21;
      v18 = *((_QWORD *)&v1 + 1);
      v3 = v25;
      v5 = v23;
      v6 = v26;
    }
    --v4;
  }
  if ( v18 > v5 )
    return 0;
  v14 = v5 - v18;
  for ( i = 0; v18 > i; ++i )
  {
    if ( v5 <= (unsigned __int64)(i + v14) )
      runtime_panicIndex(i + v14);
    v16 = *v17;
    if ( v6[i + v14].Type != *v17 )
      return 0;
    if ( (v16 == 5 || v16 == 6) && v6[i + v14].Value != v17[1] )
      return 0;
    v17 += 3;
  }
  return 1;
}
```

Yep I had no idea how to read this. But I know someone who does :).

After multiple hallucinations, wrong payloads and giving it what it wanted, it crafted a payload script for me :)

```
import struct

# Baseline sequence (Type, Value)
records = [
    (1,0), (5,3), (6,7), (2,0),
    (5,1), (6,2), (1,0), (5,6),
    (6,5), (3,0), (5,4), (6,0),
]

N = len(records)
checksum = N
for t, v in records:
    checksum ^= (t ^ v)

# All Extra fields = 0
payload = bytearray()
payload += b'\x00' * 8                  # 8 bytes ignored
payload += struct.pack('<I', N)         # count
payload += struct.pack('<I', checksum)  # checksum = 0x0000000d
for t, v in records:
    payload += struct.pack('<III', t, v, 0)

with open('payload.bin', 'wb') as f:
    f.write(payload)

print(f'Wrote payload.bin ({len(payload)} bytes)') 
```

Afterwards it just told me to run
```
curl -i --http1.1 "http://chals.tisc25.ctf.sg:57190/?t=$(date +%s%3N)" -X POST -H "Content-Type:" --data-binary @payload.bin -o flag.mp3
```

And I got the correct mp3 where the flag was spelled out letter by letter.

FLAG: TISC{PR3551NG_BUTT0N5_4ND_TURN1NG_KN0B5_4_S3CR3T_S0NG_FL4G}

Damn am I good or what :) (Read other writeups if you want to know what this chall does) (Also prepare for like 2 more writeups where CHAT just chatted :))))))))))))
