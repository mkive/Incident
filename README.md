**Category:** Forensic **Points:** 100 **Solves:** 137

> We hired somebody to gather intelligence on an enemy party. But apparently they managed to lose the secret document they extracted. They just sent us this and said we should be able to recover everything we need from it.
> Can you help?



## Write-up

For this challenge, we have a PCAP file and a Python server. It's clear that we have to recover a 'secret document' with this. Let see what we have :


### Python Server

We have a Python server which is used to communicate and execute commands over DNS, here are some interesting parts :

```python
...
from dnslib import * # dnslib
...
data = base64.b32encode(data).rstrip(b'=') # BASE32 Encoded
...
    chunk_size = 62
    for i in range(0, len(data), chunk_size): # Chunks every 62 chars
        chunks.append(data[i:i+chunk_size])
    chunks += domain.encode().split(b'.')
...
domain = 'eat-sleep-pwn-repeat.de' # Domain
...
def parse_name(label):
    return decode_b32(b''.join(label.label[:-domain.count('.')-1])) # No domain for the BASE32
...
class RemoteShell: # A little RemoteShell to execute commands and extract the secret document :)
...
```
