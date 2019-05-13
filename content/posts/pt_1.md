---
title: "Overview Of Private Trackers"
date: 2019-05-12T21:52:00-04:00
draft: false
tags: ["torrents"]
---

# Private BitTorrent trackers

A BitTorrent tracker is the software in charge of orchestrating the communication between peers that are 
using the protocol.
It keeps track of statistics and, in the case of private trackers, manages download quotas
and various other restrictions.


<div class="box">
<p class="box-title">BitTorrent Tracker software</p>
You can find a short list of open-source torrent tracker platforms 
<a href="https://web.archive.org/web/https://github.com/HDVinnie/Torrent-Tracker-Platforms">here</a>.
</div>



It’s not immediately obvious as to why private trackers would be worth the effort to
join.
For one, private torrent trackers usually have a much longer retention rate, have more organised
 content and tend to be safer than public trackers (e.g. ThePirateBay, rutracker,
rarbg).
More importantly, some private trackers focus on old/rare content that is no longer
obtainable legally.
In the next section, we will explore the general registration process of private trackers.

# Getting into private trackers

Most "top-tier" or specialized torrent trackers are *invite-only*, meaning that one has to
be invited by a current member that has reached the user class required to invite other
members. In most cases, the invite giver is responsible for the invitee’s actions. When a
user is caught trading or selling invites, his entire invite tree will be banned. Therefore,
members are very careful with their invites. This is especially true for trackers that are
deeper in the Invite map. Such an environment makes it quite unlikely for a stranger to
get invited to these trackers. Staff members also have various other countermeasures to
fight invite trading/buying (more on that in the following section). It is, however, very
possible for a person completely foreign to the torrenting world to join any tracker given
enough time and energy.

Before attempting to join a private tracker, make sure that you are familiar with the
BitTorrent protocol and the concept of sharing ratio.
Having a good <sup>upload</sup>&frasl;<sub>download</sub> ratio
is essential to keep your account at a given tracker and to climb up the user classes.
Network bandwidth is really not as important as most people think.
Dedicated or otherwise constantly running hardware and plenty of disk space – and it should not be
a surprise for seasoned data archivists – are much more valuable in the long term.
The second step is to join "open" trackers. Keep an eye on discussion boards for open-signup
 or application timeframes. Some trackers (notably, *RED*[^1] and *MAM* ) have IRC interviews.

<div class="box">
<p class="box-title">Staying Informed</p>
You can find miscellaneous information on Reddit (/r/trackers, /r/OpenSignups/)
and 4chan (/ptg/). However do not expect much help for anything beyond technical
problems coming from either forum, as most of their members are highly
paranoid. As we will see in the next section, you are mostly on your own when it comes 
to information about specific trackers.
</div>


# Invite map 

{{< figure src="/pt/map.png" title="Private trackers invite map">}}

 
The tracker invite map is an interactive visualization of the official invite routes of 
a subset of the private trackers. Most private trackers have official recruiters with an 
infinite number of invites available for members with the required user class (see figure below) 
Data was gathered manually from tracker invite forums. An arrow pointing to another 
tracker indicates that one can be invited to the tracker via an official invite thread. 
Node size indicates the approximate number of enabled users (This metric is not always 
made available). Of course this is not a complete map, partly because this information 
is voluntarily made hard to find. Trackers with no invite threads from and to other 
trackers are not shown on this map.

{{< figure src="/pt/invite_thread.png" title="Official recruiter in an invite forum">}}

To view the most up-to-date graph, raw data and source code of this project, you can
visit https://dataarchivist.net/trackermap/.
If you carefully review the tracker map, you will notice that almost all of them are
accessible through a handful of important trackers. Most notably, the music tracker
*Redacted* (RED), which one can join directly via the interview process, grants you access
to almost all other music trackers and many more.
Once you joined the "lower tier" trackers, your goal is to reach the user class that gives
you access to the invite forums. This process is different for every tracker but it always
involves either reaching a particular amount of data uploaded while maintaining a good
ratio, uploading a number of new torrents or staying a member for a certain length of
time. There are various guides for specific trackers linked in the Additional Reading
section.

# Rules, "Marking" and security

## Marking

It is generally frowned upon to share information relating to a tracker in a public forum.
As such, staff members, are supposedly on the lookout for data that could be linked to
a specific member – a screenshot of a forum which would indicate the date at which a
certain thread was last visited, an exact ratio or a specific number of bonus points, etc.
– and would in turn flag them. Flagged, or */marked/* members are more likely to get
banned for minor offenses.

{{< figure src="/pt/4chan_marked.png" title="Links posted in /ptg/ are seen as marking attempts by staff.">}}

## Security

Although you are a lot less likely to receive copyright notices from your ISP while using a
private tracker, it is generally a good idea to assume that the private information related
to your accounts (your email, username password and IP address) is **not** safe. It is good
practice to use different usernames and try to avoid making them associable (avoid
sending screenshots of your other accounts[^2]). Always use strong, unique passwords for
all your accounts. Do not share any personal information (or any information at all, if
possible) in the forums or IRC channels, and do not bother with trackers that are asking
for personal information during the registration process.

# Additionnal reading

Sinderalla database leak

* [4chan message #1](https://web.archive.org/web/20190101190608/https://rbt.asia/g/thread/66466024/#q66471906)
* [4chan message #2](https://web.archive.org/web/20190101190608/https://rbt.asia/g/thread/66466024/#q66472549)
* [4chan message #3](https://web.archive.org/web/20190101190608/https://rbt.asia/g/thread/66466024/#q66485228)
* [Reddit discussion](https://www.reddit.com/r/trackers/comments/8tg5pt/sinderella_database_being_leaked/)

IPTorrents owner dox (read the Why should I care section)

* [https://iptorrentsdox.com/](https://web.archive.org/web/20141026015031/https://iptorrentsdox.com/)

/ptg/ 

* Wiki [https://wiki.installgentoo.com/index.php/Private_trackers](https://wiki.installgentoo.com/index.php/Private_trackers)
* FAQ [https://pastebin.com/thLgSkNE](https://web.archive.org/web/20190504103634/https://pastebin.com/thLgSkNE)

[^1]: You can find a walkthrough of the interview process [here](https://web.archive.org/web/20181230024615/https://wiki.installgentoo.com/index.php/Redacted.ch)
[^2]: It is common practice to ask for ratio proofs when invinting other members.

