---
title: "Running a LibGen cache server"
date: 2020-02-20T15:39:04-05:00
author: simon987
tags: [torrents]
---

# nginx proxy cache
Not too long ago, there was [an initiative](https://www.vice.com/en_us/article/pa7jxb/archivists-are-trying-to-make-sure-a-pirate-bay-of-science-never-goes-down) to secure the books and scientific papers of the *Library Genesis* project. 
It attracted a lot of new seeders and project contributors, however, I noticed that the daily database dumps were becoming
slower and slower to download because of the increased traffic.


{{< figure src="/lg/curl1.png" title="43kBit/s from origin server">}}

I decided to try to contribute some of my bandwidth to the project by creating a mirror of the database dump files.
The initial idea was to write a bash script that would periodically download all new dumps, clean up the old ones and 
somehow handle connection problems and duplicate files. I quickly realized that this solution could become a hassle to
maintain so I opted for a simpler alternative.


## Basic nginx setup

The following configuration is all that is needed to get a cache server up and running.

{{<highlight nginx >}}
proxy_cache_path /files/.cache/ 
	levels=1:2 
	keys_zone=libgen_cache:1m
	max_size=90g inactive=72h use_temp_path=off;


location / {
	proxy_cache libgen_cache;
	
	proxy_ignore_headers X-Accel-Expires Expires Cache-Control;
	proxy_cache_valid any 168h;
	proxy_cache_revalidate on;
	
	add_header X-Cache-Status $upstream_cache_status;
	proxy_pass http://gen.lib.rus.ec/dbdumps/;
}
{{</highlight>}}

The `proxy_cache_path` statement initializes a 90 GB cache folder. Entries that are not
accessed for more than 72h are periodically purged. 
See [ngx\_http\_proxy\_module](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache_path) 
for more details about all its options.

In the `location` block, we tell nginx to ignore the client's headers and to consider all cached
items valid for (an arbitrary value of) one week. After 168 hours, a file is considered *stale*,
but it will still be served from cache if it wasn't modified on the origin server (using the `If-Modified-Since` header).

The `$upstream_cache_status` variable tells the client if they're downloading from
the origin server or from the cache.

{{< figure src="/lg/cachehit.png" title="X-Cache-Status header">}}

## Download speed improvements

The initial download for `libgen.rar` took 3h12m (~310 kBit/s). When I re-downloaded the file immediately after,
I was able to saturate my home connection and finish the download in 3 minutes, about 60 times faster!

You can find the cache server at [lgmirror.simon987.net](https://lgmirror.simon987.net/).

{{<highlight "" >}}
Connecting to lgmirror.simon987.net (lgmirror.simon987.net)|104.31.86.142|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3431679578 (3.2G) [application/x-rar-compressed]
Saving to: ‘libgen_2020-02-17.rar’
2020-02-20 18:16:22 (13.9 MB/s) - ‘libgen_2020-02-17.rar’ saved [3431679578/3431679578]
{{</highlight>}}


### Limitations and workarounds

I noticed that the file listing has shortcuts pointing to the latest database dump.
 Unfortunately, due to the way nginx's `proxy_cache` module works, both files would need
to be pulled from the origin server, even if they are identical.

{{< figure src="/lg/dbdump.png" title="libgen.rar is symlinked to today's dump">}}

Since I'm not aware of a way to create a HTTP redirect based on the current date,
a workaround for now is to force users to use the `*_yyyy-mm-dd.rar` files.

{{<highlight nginx >}}
location /libgen.rar {
	add_header Content-Type "text/plain;charset=UTF-8";
	return 200 'Please download libgen-mm-dd.rar instead.\nПожалуйста, скачайте libgen_yyyy-mm-dd.rar.\n';
}
{{</highlight>}}

## Full configuration

<noscript>
	<a href="https://gist.github.com/simon987/40f8d81878c45e43a6b91db327d8f4c0#file-libgen_mirror-conf">libgen_mirror.conf</a>
</noscript>

<script src="https://gist.github.com/simon987/40f8d81878c45e43a6b91db327d8f4c0.js"></script>
