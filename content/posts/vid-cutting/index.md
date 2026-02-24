---
title: "Journey for the (almost) lossless video cutting software"
date: 2026-01-30T08:06:25+06:00
description: Chronicling my dillema in trying to get frame-perfect cuts on videos and attempting to build a smart-cutting solution
menu:
  sidebar:
    name: Lossless Video Clips
    identifier: vid-cut
    weight: 30
author:
  name: Nathan Henderson
  image: /images/author/nathan.png
tags: ["Video", "FFMPEG"]
categories: ["Multimedia"]
---
Ever since I first discovered [ErsatzTV](https://github.com/ErsatzTV/ErsatzTV), the FOSS that allows you to host your own IPTV network, I'd become fascinated with building up a collection commercials, or 'filler' as Ersatz calls them, to play in-between content. To begin with I looked for Blu-Rays and DVDs I owned that contained TV Spots as special features (some didnt even advertise TV Spots and just called the 30 second spot a 'trailer') but these are few and far between. Plus, you don't want all of the ads to just be of movies so my attention then turned to YouTube. Most of these are 30+ minute long broadcast recordings so I began my search for ways to trim these down into individual files...

---
## 1. Premiere Pro and the beauty of D-VHS
I'd dabbled with YouTube in a variety of ways over the years, whether editing friends videos or some of my own so I was quite familiar in Adobe's Premiere Pro. At the time this seemed like the most logical solution, just chuck the downloaded YouTube video into a new Project and cut the commercials out one-by-one. It was a long and tedious but it worked. Sure, the quality wasn't great but I just wrote that down to the poor quality of the VHS source. That was, until, I stumbled across a goldmine - the D-VHS tape. I can write a whole other blog post on these things but you can get quick summary of them [here via TechMoan](https://www.youtube.com/watch?v=jiu0LPeLQPE).

The video goes into Movies released on the format but they were treated like regular VHS tapes so those who had the money and were so inclined were about to record a clean 1080i signal from their TV in the early 2000s. And there is a singular legend who is not only ripping D-VHS tapes he finds onto sites like YouTube but also providing the *raw data* onto the Internet Archive. [Here](https://archive.org/details/wrtvsuperbowlxl2506) is an example - it is a 21.8GB .ts file that shows a clean and crisp recording of the 2006 Superbowl. Although I'm not one for NFL, in terms of getting interesting ads in the best quality possible this was an absolute revalation! Access to broadcast recordings this high quality from this time period is rare and it acted as a double-edged sword as it laid bare the flaw in my clip creation process. I'd soon learn the golden rule - *never compress from a compressed source*.

Seeing how clean and crisp the footage was, I had to find a way to splice the commercials out in a way that wouldn't require compression. Sure, I could tweak and tune the settings to get it just right but i'm a bit of a perfectionist and would have rathered keep the commercial files in their purest form. As such, I began searching for ways to cut segments out from a video without having to re-encode. Enter: Lossless Cut.

---
## 2. Lossless Cut
[Lossless Cut](https://github.com/mifi/lossless-cut) is a fanastic tool, being a self-described "swiss army" knife of video editing. I have to say it is quite flexible, being a cross-platform app that runs low on resources but is fast, efficient and is seemingly perfectly built for my usecase. You simply add the video onto the tool and you can easily create a new "segment" for every clip you want to make. For me, that simply meant a new ad - and on top of that, I could still do the frame-by-frame navigation to make sure the clip started at the exact point the screen went black so all of the files I was creating wouldn't carry anything from the advert before it. I was very excited to find this open-source tool that seemed perfectly built for me! Not only was it easier to map out the advertisements that I wanted as opposed to manually doing them one-by-one in Premiere Pro, but LosslessCut proclaimed to create these cuts without re-coding, so it retained all of the original video quality! 

Mind you, if it was that simple then I wouldn't be here writing this post. After making my first run through the app I soon realised an issue. No matter how precise my segment times were, the start of some of the ads always retained a second or two from the previous ad. Initially, I thought that I was doing something wrong so I played around with the advanced settings but couldn't seem to fix the issue. I saw there was a "Smart Cut" option that proclaimed to fix cutting issues but it never worked correctly, instead resulting in all of my clips to repeat themselves for a few seconds. That's when I learnt about "keyframes".

---
## 3. How does Video Compression work?
Videos are, at the end of the day, a collection of images. We all understand this conceptually, but what it actually means is videos are data hungry. Lets take a typical 30 second commercial at the NTSC broadcast rate of 29.97 FPS. That means you'd have about 899 images making up that video. Given that the NTSC standard has a resolution of 480p, an uncompressed full-data image comes to [about 27MB](https://www.cl.cam.ac.uk/~jac22/books/mm/book/node111.html#:~:text=Table_title:%20How%20Big%20Is%20a%20Single%20Frame,%7C%20640x480:%2097Tb%20%7C%20320x240:%2024Tb%20%7C), which would result in that 30 second 480p commercial being 810MB in size. Sure, we could store that find nowadays but by no means is that efficient. That's why we have video compression and where these "keyframes" come into play. I'm by no means an expert, this is a hobby venture for me, but my understanding is that no matter the compression method applied to the frame the general concept remains the same.

You are trying to cut down on the video information that is stored on a frame-by-frame basis and this results in your video being made up of 3 frame types
1. I-Frames
2. P-Frames
3. B-Frames

Your I-Frame is a complete image. It may go through some form of compression from its pure uncompressed form but this frame will stand on its own, it is not dependant on another frame but rather will become the basis for the next two frame types.

The P-Frame is known as a Predicted Picture, as it stores the information that relate to the changes from the previous frame. As such, this frame has partial information about the video because it usually includes the smallest about of data possible.

The B-Frame is a Bi-Directional Predicted Picture and works in a similar manner to your P-Frame, however this has the potential to store the differences between the current frame as well as the following frame - creating further opportunities for storage space.

The Wikipedia page for [Video Compression Types](https://en.wikipedia.org/wiki/Video_compression_picture_types) has this great infographic that demonstrates this in action


{{< img src="https://upload.wikimedia.org/wikipedia/commons/6/64/I_P_and_B_frames.svg" align="center" title="Pac-Man Frame Infographic">}}

Again, the way these frames are handled and created will vary wildly depending on the compression algorithm you are using but at a low-level, this is a (very generalised) overview of Video Compression. However, this gives us enough information to understand what's going on!

---
## 4. 'Smart' cutting
As you might have already guessed, the 'I-Frame' is the 'keyframe'. Since it is not dependant on any previous frames and can stand on its own, tools like FFMPEG are able to create a new video starting from that exact point and continuing on. However, if the place you wish to cut from is on a P/B frame then that is simply impossible and the best option you have is to either re-encode from that point or go back to the most recent I-Frame. Although I found deep satisfaction from being able to identify exactly what was going on, and a greater appreciation for the process as a whole I found myself depleated. This is just how videos are made and there's not much you can do - you HAVE to perform some sort of re-encoding if you want the video to start at the exact position you want it to... but does that mean you have to re-encode the entire video?

The understanding of the process gave me an idea - if videos nowadays is just made up of I-Frame segments, what if we re-encoded up to the next I-Frame? That is, instead of re-encoding the entire video past the problematic cut, why couldn't you just re-encode those first few seconds so the start point is correct and once you reach the next I-Frame, splice the video and append the remaining uncompressed video onto the re-encoded clip? From the looks of it, it seems as though this is exactly what Lossless Cut's Smart Encode is attempting to do but like I said, I'd never had any actual luck with the process. Figured it was worth a shot, right? 

---
## 5. Custom GO Script
To embark on this new journey, I downloaded a different tool that's become a stable amongst multimedia enthusiasts - 'avidemux'. A fantastic tool (if a little outdated) that allows you to quickly jump not only frame-by-frame but also keyframe-by-keyframe and it also shows you the type of frame you are looking at. I took a clip i'd pulled from a D-VHS source tape and found that the true start of the commercial was about 0.15 seconds into the clip. Then, I jumped into GO and started throwing together a quick little script that used FFMPEG wrappers like ffmpeg-go and ffprobe-go. FFProbe was first used to get information about the next closest keyframe from the given cut point. Unfortunetly, ffprobe-go didn't include any frame information so I forked it and build the structs myself based on the terminal output I was getting, which has since been added into the library. Once we had that keyframe time, we passed it into FFMpeg to build up the two segments.

Because we are trying to append these two segments together its important that the first re-encoded segment matches the source video near identically. So, we needed to pull information about the codec as well as bitrate information. The bitrate won't affect our ability to append, but I found that with the D-VHS source having a mpeg2video codec, FFMPEG's default is super blocky so I just told it to match the source's max bitrate. These segments were generated just fine, cutting out the initial information I wanted to leave out and ending right at the following keyframe.

Next, we had to take that keyframe timecode and create a secondary segment that continued onwards from the keyframe, but ensuring we are copying the video out so that no video information was lost. Its a simple operation so it didn't require too much work so once these two segments were generated I quickly went ahead and tried to append them together and ... it kind of worked? The vision was there, you could see that the commercial started exactly when I needed it to and visually it flowed nicely with only a second or two of blocky compression artifacts but it would stutter or freeze at the join point. I did some research and it turns out that although i'd been using the timecode the keyframe was associated with, because this was a raw D-VHS broadcast tape it included all the broadcast metadata associated with that which is reliant on syncing up the video with the audio, as well as making sure its staying on time. By forcibly splicing part of the video away and creating a new video that was seconds in but acting as though the start time would be 0, it created this broken mismatch of the timecodes.

FFMPEG can handle this for you, however, by simply massing through flags like +ggpts to re-generate the timecodes as well as "-avoid_negative_ts make_zero" to shift the timestamps.