---
title: "Automating Youtube Archival"
date: 2019-05-11
draft: false
tags: ["youtube-dl", "automation"]
---

Google has been known to terminate entire Youtube channels without 
notice in the hope of staying advertiser friendly. **58 million "problematic" videos
were deleted from the platform in Q3 2018** <sup>[1](#sources)</sup>.
This week we are exploring various Youtube archival solutions that utilizes
 [youtube-dl](https://github.com/ytdl-org/youtube-dl/),
 a command-line program that extracts and downloads videos from web pages.

{{< figure src="/ytdl/1.png" title="Channels removed, by removal reason">}}



# Installation

Install youtube-dl via pip to ensure that you have the latest version. Google often
pushes updates that breaks youtube-dl so you always want to stay up to date. You
might be able to install it via your distibution’s package manager but it’s often several
versions behind.

```
pip install --upgrade youtube-dl
youtube-dl --version
```

# Basic usage

You can now use the youtube-dl command to download a single video, a playlist or a channel:

```
youtube-dl https://www.youtube.com/watch?v=XXXXXXXXXX
youtube-dl https://www.youtube.com/channel/XXXXXXXXXX
```

You can use the same command to download videos from various websites.Youtube-dl
is also a Python library that you can use in scripts:

{{<highlight python "linenos=table">}}
#!/usr/bin/python

import youtube_dl

yt_dl = youtube_dl.YoutubeDL()
yt_dl.download("https://www.youtube.com/watch?v=XXXXXXXXXXXX")
{{</highlight>}}

## Command line arguments & Scripting

This document is not a replacement for youtube-dl’s documentation, you can find the
updated list of command line arguments on its [Github page](https://github.com/ytdl-org/youtube-dl/).

Below is a bash script that will download everything specified in `list.txt`, a text file
with a youtube channel or video on each line. The script will save the URLs of the videos
in archive.txt as it downloads them to speed-up the subsequent executions.
The `--write-info-json` and `--write-thumbnail` options ensures that we also download
the video metadata such as the description and the title.

{{<highlight Bash "linenos=table">}}
youtube-dl -a list.txt -o '%(uploader)s/%(title)s-%(id)s.%(ext)s' \
	--write-thumbnail\
	-f "bestvideo[ext=webm]+bestaudio[ext=webm]/bestvideo[ext=mp4]+bestaudio[ext=m4a]/bestvideo+bestaudio/best"\
	--write-info-json\
	--geo-bypass\
	--ignore-errors\
	--download-archive archive.txt
{{</highlight>}}

You can decide to run the script periodically or to schedule it with cron:

```
crontab -e
```

{{<highlight Bash>}}
# Download new videos every 2 hours
0 */2 * * *
/mnt/Archive/Youtube/archive_yt.sh
{{</highlight>}}

## Live streams

While a cron job will download all videos uploaded by a channel (if the uploader does
not delete the video between executions), it does not handle live streams. Youtube-dl
allows you to download live streams with the same command but you obviously have to
start the execution during the stream.
Below is a different approach that takes advantage of the Youtube email notification
feature. This simple Python script reads your last 3 emails and searches for a youtube
link in the email body. It will immediately start downloading the video using the youtube-dl
Python library. You can use this method to download uploaded videos as well as live
streams. If you are using a gmail account, you will need to genrate an [App Password](https://security.google.com/settings/security/apppasswords)
 to allow the script to login.

{{<highlight python "linenos=table">}}
#!/usr/bin/python
  
import imaplib
import re
import youtube_dl

# Initalize the youtube-dl downloader, nooverwrites param will
# skip videos that are already downloaded or 
# currently being downloaded
yt_dl = youtube_dl.YoutubeDL(params={
    "nooverwrites": True,
    "nopart": True,
    "format": "bestvideo[ext=webm]+bestaudio[ext=webm]/bestvideo[ext=mp4]+bestaudio[ext=m4a]/bestvideo+bestaudio/best"
    })

# This regex pattern matches youtube video links
YT_LINK = re.compile("Fv%3D([^%]*)%")

mail = imaplib.IMAP4_SSL("imap.gmail.com")
mail.login("username@gmail.com", "password")
mail.list()
mail.select("INBOX")

# Fetch the last 3 emails
_, data = mail.search(None, "ALL")
last_emails = list(reversed(data[0].split()))[:3]

for num in last_emails:
    _, data = mail.fetch(num, "(RFC822)")
    body = data[0][1].decode()

    # Check for pattern match
    match = YT_LINK.search(body)
    if match:
        url = "https://youtube.com/watch?v=" + match.group(1)
        yt_dl.download([url, ]) # immediately download
{{</highlight>}}


## Automatically upload to rclone remote

To take advantage of cloud storage, you can setup [ytdlrc](http://github.com/bardisty/ytdlrc) to automatically move
videos to an rclone remote as they are downloaded. This simple script is completely interchangable with
youtube-dl and can be setup on a machine with low disk space.
The script uses your existing youtube-dl and rclone configuration and is ideal for
setting up automatic Youtube archival on a cheap VPS.

## Archiving Metadata

If you wish to save a video’s metadata without downloading the actual video, there are command line utilities dedicated to this task.

* [Youtube-MA](https://github.com/CorentinB/YouTube-MA)
* [yt-mango](https://github.com/terorie/yt-mango)


---- 

# Sources

* https://transparencyreport.google.com/youtube-policy/removals


