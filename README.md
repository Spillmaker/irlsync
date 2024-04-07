# IRLSync - Synchronize your independently broadcasted video-feeds.

(TLDR: This is not the solution you are looking for. yet... But this is an attempt on collecting all the info regarding this topic, so that we one day can have a proper solution.)


This is a Work In Progress, have anything to contribute? Create a Issue or PR or find me somewhere on discord (spillmaker)


## The Problem
IRL streaming is the action of bringing your stream out into real life. Either by a phone-apps like Moblin that broadcasts the videofeed from the camera, professionally made broadcasting encoders from manufacturers like TVU and LiveU that takes the feed from one or more cameras, or homemade backpacks like Belabox and IRLbox.

One thing all of these solutions has in common, is the challenge to overcome the nature of celluar networking. Solving issues like congestion, signal noise, poor signal, different operators and all of this combined in a moving scenario.
The common solution has been to use multiple celluar connectioins and spreading the video feed across them all. This concept is referred to by many names and terms. Like Link Bonding, Link Aggregation, and protocls like SRTLA (Belabox), RISTLA (?)(IRLBOX), LRT (LiveU) and Inverse StatMux Plus (TVU)

These solutions are starting to get pretty mature, and link bonding is starting to be a non-concern as there are multiple free ways of doing this now. For most people this is sufficient, but for people who want to take IRL-streaming to the next level, there is one big issue.

Video-feed syncing.

For people who can afford a LiveU Backpack or the "Contact us" prices of TVU Encoders there are solutions for this available. (Need to confirm if its possible with LiveU systems) The professionally made backpacks implement timecodes, which the recieving ling bonding server forwards to the downlink, where multiple feeds can be synced within the milisecond of eachother.

For more "prosumer" encoders like Belabox and IRLBox, this is not a feature that is supported. It is uncertain to why, but my theory is that this is a "chicken and egg" scenario where the broadcasters are waiting for support in the recievers, and the recievers dont care because there is no support in the broadcasters.

The IRL-steaming category is around 5000 users [^1] so its pretty safe to say its a niche market. The most used broadcasting software, OBS has a lot of things to implement and fix, and adding support for timecodes is probably not too high on their list since its not too many people requesting it. 

So if IRL-streamers want something done, its often best to do it ourselves.

![thanos-do-it-myself](https://github.com/Spillmaker/irlsync/assets/696120/933ab0d0-66fa-4a7f-b7af-80565fcfd42b)

## Situation Report
Since the problem can be considered a evil circle, the root of the problem could be anywhere, but for this project, i consider the root of the issue to be OBS.

From my experience OBS have to main issues
- It does not have support for timecodes in media from network-protocols
- It (appears to) load Media-sources consecutively and not simoultanesouly.

The first problem is the most obvious one. Withouht support for timecodes, there is no way to force the video-feeds into sync.
The second problem makes this all more difficult, because even if you managed to generate network-feeds (RTMP, SRT) which gets synced and then fed into OBS, they will still not be synced because OBS does not load them at the same time!

We will come back to solving the first issue, but lets take a look at the second issue first.

## Solution to Media Sources - Dont use them.
The best solution to keeping media sources in OBS in sync is simply to not use them. Form experiments it appears that using multiple capture cards, they are loaded unbuffered so even if they load consecutively, their feed will still be in sync.
So for this project, to solve the issue of timecodes, i will be buildng software that can be installed in single board computers which we will be use to feed HDMI into capture cards then into OBS.


(Allegedly this can also be solved with NDI, but more tests are needed)

## Solution to respecting timecodes - Use a downling backbone.
I dont know what this is called in the professional world (Anyone?) but manufacturers like TVU and LiveU offers these magical servers that you can have in-house and recieve the downlink directly. What they implemented here as well, was the syncinc of the timecodes, and possibility to output them completely synced either trough HDMI or the more professional cable, SDI.

Basicly the goal is to make our own server like this. A "Contact us" budger server, but now in the cheaper budget.

I am still working on the Proof of Concept of this, but for now i am basing this on using known single board computers like the Jetson Nano (EOL) and single board computers based on the Rockchip RK35xx chipset. Both which are popular among the free encoder market. (Belabox, IRLbox) Because of the nature of how this work, i think i will also be able to make images for certain Raspberry Pi boards as well. (As long as the board can handle to decode a incoming stream and outputting to HDMI, it should in theory work)

(This is where my work is at this point.)


## Future - Agreeing on the protocol for the timecodes.
In the IRL-streamer communities its often reffered to so called SEI time-codes. Basicly this is a way of stamping a timecode into the frame-data of the container of the videostream. Ive barely started the research on this, but ive discovered that gstreamer. (Which is used by atleast Belabox to broadcast) supports a pipeline-element to stamp a timecode in.

For this project ive decided to work out from using RTC for the timestamp. So for gstreamer the element would be:

```timecodestamper source="rtc"```

This would add (how?) a timecode into each frame (what protocol?) with the local machines time. ```HH:MM:SS:MSMS``` for example ```23:59:59:99```

So by adding this element in the pipeline of the broadcasting end, the goal is then to be able to read it and hold the stream until the recieved timecode is approached, then output the video-feed to HDMI. This would mean that as long as all the machines in the IRL-setup are using the same NTP-server, the outputted HDMI-feeds should all be within 1ms of eachother!

I am not sure if this would be the definite way of doing it, since there are other "protocols" for adding timecodes, and im not sure if this is using the SEI timekode format as people are referring to. But basing this on Gstreamer, one of the two most used libraries used for emitting a videofeed over the network (Other being ffmpeg) i think its a safe place to start.

Do you have any knowledge around this? I would love to hear from you, Make a ticket, a PR with a new file where you add your take on this, or find me in one of the IRL-community discord around there or try to DM me on discord. (Spillmaker)

- Krister Berntsen AKA Spillmaker

[^1]: https://tenor.com/view/armstrong-made-it-up-gif-25653819
