---
title: "TryHackMe: OhSINT - Writeup"
date: 2019-06-20 00:00:00 +0300
categories: [cybersec, infosec, osint, tryhackme]
tags: [osint, open source intelligence, intelligence gathering]
---

## Information
OhSINT is a fairly simple box that is available on TryHackMe with its main goal is to introduce OSINT and the tools 
that are used in the field to those who are curious about it. The box has straightforward tasks based on an altered 
default Windows XP wallpaper.

## Tasks
- Find the avatar of the owner of the picture
- Find the city that he has been on holiday
- Find his password
- Find the city that he lives in
- Find the `SSID` of the WAP he is connected to
- Find his email address
 
## Quick Summary
- Use `exiftool` to extract the metadata from the given picture
- Google the name from `exiftool` to find the accounts associated with it
- Run the given `BSSID` in [Wigle][Wigle]
- Enumerate the given username for possible alternative accounts
 
## Tools Used
- exiftool
- Google
- [Wigle][Wigle]

## Walkthrough
### exiftool
Let's take a look at our image first.

![Windows XP][Windows XP]

Besides some purple grass on the left, there is nothing significant in terms of information here. Let's examine its 
metadata with `exiftool`.

```
exiftool WindowsXP.jpg

ExifTool Version Number         : 11.16
File Name                       : WindowsXP.jpg
Directory                       : .
File Size                       : 229 kB
File Modification Date/Time     : 2019:06:08 11:42:31-04:00
File Access Date/Time           : 2019:06:08 11:42:31-04:00
File Inode Change Date/Time     : 2019:06:08 11:42:44-04:00
File Permissions                : rwxrwx---
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
XMP Toolkit                     : Image::ExifTool 11.27
GPS Latitude                    : 54 deg 17' 41.27" N
GPS Longitude                   : 2 deg 15' 1.33" W
Copyright                       : OWoodflint
Image Width                     : 1920
Image Height                    : 1080
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)
GPS Latitude Ref                : North
GPS Longitude Ref               : West
Image Size                      : 1920x1080
Megapixels                      : 2.1
GPS Position                    : 54 deg 17' 41.27" N, 2 deg 15' 1.33" W
```

Our image belongs to a person named `OWoodflint`.

### Data Gathering
Googling `OWoodflint` reveals several results. Most notable results are a [Wordpress blog][Wordpress 1] and a 
[Twitter][Twitter 1] account.

![Google 1][Google 1]

#### Wordpress
If we take a look at the Wordpress blog we'll see a single post.

![Wordpress][Wordpress 2]

We now know that he is in New York right now. Additionally, if you are acquainted with Wordpress, you'd know that the 
post info (poster, category, date etc.) comes right after the text, but in this instance that there is a gap. If we 
highlight the text we'll see a 'hidden' text, reading `pennYDr0pper.!`; which is our password.

![Wordpress][Wordpress 3]

If you are not acquainted with Wordpress you can also check the source code of the page. Specifically the area of the 
code where the post begins:

```html
<article id="post-3" class="post-3 post type-post status-publish format-standard hentry category-uncategorised">

[...]
        
<p>Im in New York right now, so I will update this site right away with new photos!</p>

<p style="color:#ffffff;" class="has-text-color">pennYDr0pper.!</p>

[...]

</article>
```

#### Twitter
If we take a look at the Twitter we'll see that his avatar is a picture of a cat and two tweets:

![Twitter][Twitter 2]

The first tweet mentions a `BSSID` of `B4:5D:50:AA:86:41`. 

#### Wigle
If we take the `BSSID` and enter it into [Wigle][Wigle], we'll see where this router is alongside its `SSID`. In this 
case its in London and its `SSID` is `UnileverWiFi`.

![Wigle 1][Wigle 1]

![Wigle 2][Wigle 2]

#### Enumerating the Username
Since we haven't found anything regarding his email we'll have to enumerate the username to see if anything pops up. 
Substituting letters with numbers tend to work well. In this case `Owoodflint` becomes `OWoodfl1nt` and searching for 
`OWoodfl1nt` reveals a Github profile.

![Google 2][Google 2]

#### Github
Taking a look at the Github profile, we'll see that it has a single repository and no additional info on his profile. 

![Github 1][Github 1]

Taking a look at the repo `people_finder` we'll only `README.md`, which reads as follows:

```markdown
# people_finder

Hi all, I am from London, I like taking photos and open source projects. 

Follow me on twitter: @OWoodflint

This project is a new social network for taking photos in your home town.

Project starting soon! Email me if you want to help out: OWoodflint@gmail.com
```

![Github 2][Github 2]

We know that his email address is `OWoodlint@gmail.com`. By collecting this last piece of information, we've collected 
all the flags and finished this box.

[Windows XP]:       /images/posts/2019-06-20-thm-ohsint/WindowsXP.jpg
[Wigle]:            http://wigle.net/map
[Wordpress 1]:      https://oliverwoodflint.wordpress.com/author/owoodflint/
[Twitter 1]:        https://twitter.com/owoodflint
[Google 1]:         /images/posts/2019-06-20-thm-ohsint/Google%201.png
[Wordpress 2]:      /images/posts/2019-06-20-thm-ohsint/Wordpress%201.png
[Wordpress 3]:      /images/posts/2019-06-20-thm-ohsint/Wordpress%202.png
[Twitter 2]:        /images/posts/2019-06-20-thm-ohsint/Twitter.png
[Wigle 1]:          /images/posts/2019-06-20-thm-ohsint/Wigle%201.png
[Wigle 2]:          /images/posts/2019-06-20-thm-ohsint/Wigle%202.png
[Google 2]:         /images/posts/2019-06-20-thm-ohsint/Google%202.png
[Github 1]:         /images/posts/2019-06-20-thm-ohsint/Github%201.png
[Github 2]:         /images/posts/2019-06-20-thm-ohsint/Github%202.png
