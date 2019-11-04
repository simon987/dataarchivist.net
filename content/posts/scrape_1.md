---
title: "Web Scraping 101: E-Reader app"
date: 2019-05-13T10:46:34-04:00
tags: ["scraping"]
draft: false
author: "simon987"
---

Let's say you bought a textbook and it comes with
a code that lets you read its online version.

Of course that online version is tied to your account, it expires in 6 months and
is not compatible with your tablet browser. So what are you gonna do? Hack together a script
that takes screenshots of the pages? That's not a bad idea, but first let's see if we can
get through the e-reader's DRM.

After logging into the app, I immediately open the dev tools and this is what I see:

{{< figure src="/scrape/dev_tools1.png" title="">}}

So, individual pdf pages are being read from this `getpdfpage` endpoint, rendered with a Javascript library and displayed in your browser
every time you flip a page in the app.

This is what is sent to the endpoint:

{{<highlight javascript >}}
globalbookid: "<hash>"
pdfpage: "<hash>.pdf"
iscover: "N"
authkey: "<hash>"
hsid: "<hash>"
{{</highlight>}}

Obviously, the `globalbookid` is the unique ID of the book I am looking at. `pdfpage` is the ID of the page, there is
probably a way to get a list of those with another endpoint. `iscover` and `authkey` are self explanatory. So what exactly is
that `hsid` parameter? From what I can see, it is different for every request. 

Looking further, I find the `getpagedetails` endpoint, which does exactly what the name suggests:

{{< figure src="/scrape/dev_tools2.png" title="/getpagedetails">}}

Okay, so we have our authkey, the list of `pdfpage`s, and we know the `globalbookid`. Let's try to dig into the minified Javascript
code to find out how the `getpdfpage` endpoint is called.

{{<highlight javascript "linenos=table,linenostart=370931">}}
var o = "".concat(e.serverDetails, "/ebook/pdfplayer/getpdfpage?globalbookid=") + "".concat(e.globalBookId, "&pdfpage=").concat(t.pdfPath, "&iscover=N&authkey=").concat(r),
	i = o.replace("https", "http"),
	c = Object(s.c)(l.b.MD5_SECRET_KEY + i);
o = "".concat(o, "&hsid=").concat(c), n.pdfPath = o, a.bookPagesInfo.pages.push(n)
{{</highlight>}}

Interesting... So the query URL is built by concatenating the different parameters together as you would expect but then a part of the 
URL - everything but the mysterious `hsid` parameter - is put into a hash function and its result is the value of the `hsid` parameter.

Without even looking at the `s.c` function it is becoming more and more obvious that the value of `hsid` is an MD5 hash of the whole query URL,
with `l.b.MD5_SECRET_KEY` as the salt. 

{{< figure src="/scrape/dev_tools3.png" title="MD5_SECRET_KEY hidden in plain sight">}}

The secret code was hidden only a few keystrokes away into the source. Now that we have all the puzzle pieces,
 let's hack together a simple Python script to automate the download process: 

{{<highlight python "linenos=table,linenostart=18">}}

def get_page(page):
	# Generate the 'hsid' verification hash
    verification = hashlib.md5(("%s%s/ebook/pdfplayer/getpdfpage?globalbookid=%s&pdfpage=%s&iscover=N&authkey=%s" 
							   % (MD5_SECRET, URL, BOOKID, page["pdfPath"], AUTHKEY)).encode()).hexdigest()

    r = requests.get("%s/ebook/pdfplayer/getpdfpage?globalbookid=%s&pdfpage=%s&iscover=N&authkey=%s&hsid=%s"
                     % (URL, BOOKID, page["pdfPath"], AUTHKEY, verification, ))

    print(r.status_code)

	# Write the raw pdf response to a file
    with open(BOOKID + "_" + str(page["pageOrder"]) + "_" + page["bookPageNumber"] + ".pdf", "wb") as out:
        out.write(r.content)

# To save time, I manually saved the content of /getpagedetails to file
with open("book.json") as f:
    for page in json.load(f)[0]["pdfPlayerPageInfoTOList"]:
        get_page(page)

{{</highlight>}}

To stitch the pages together, I used `pdfunite`:

{{<highlight bash >}}
pdfunite $(ls -v) output.pdf
{{</highlight>}}

Now even if you wanted, you *couldn't even buy* a digital version of that book of that quality.
