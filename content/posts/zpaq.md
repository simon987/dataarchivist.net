---
title: "Android phone backups with zpaq"
date: 2019-11-05T13:16:27-05:00
draft: true
tags: ["backup"]
author: simon987
---

{{< figure src="/zpaq/10gb.png" title="Benchmark for 10GB">}}

{{<highlight bash>}}
pkg install g++ make git
git clone "https://github.com/zpaq/zpaq"
cd zpaq/
# zpaq must be compiled with -DNOJIT for non-x86 processors
g++ -Ofast -DNOJIT -Dunix zpaq.cpp libzpaq.cpp -pthread -o zpaq
{{</highlight>}}

Initial backup can take a while to complete, 
{{<highlight _>}}
$ zpaq add "arc???" ./files/ -index local-index.zpaq
0.000000 + (955.283380 -> 687.840444 -> 622.268166) = 622.268166 MB
45.737 seconds (all OK)
{{</highlight>}}

But subsequent ones are almost instantaneous if no files were changed
{{<highlight _>}}
$ zpaq add "arc???" ./files/ -index local-index.zpaq
0.000000 + (0.000000 -> 0.000000 -> 0.000104) = 0.000104 MB
0.408 seconds (all OK)


## 
$ ls -lh
total 594M
-rw------- 1 u0_a94 u0_a94    594M Nov  5 14:18 arc001.zpaq
-rw------- 1 u0_a94 u0_a94     104 Nov  5 14:18 arc002.zpaq
-rwx------ 1 u0_a94 u0_a94     362 Nov  5 14:17 backup.sh
-rw------- 1 u0_a94 u0_a94    411K Nov  5 14:18 local-index.zpaq
{{</highlight>}}

{{<highlight bash "linenos=table">}}
#!/usr/bin/env bash
  
zpaq add "arc???" \
        ~/storage/shared/DCIM \
        ~/storage/shared/Documents \
        ~/storage/shared/Download \
        #...
        -index local-index.zpaq -m2

rclone move arc*.zpaq my-remote:/backups
{{</highlight>}}

