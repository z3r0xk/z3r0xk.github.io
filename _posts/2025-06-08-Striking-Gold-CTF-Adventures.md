---
title: "Striking Gold: My Adventures in Goldrush Gauntlet CTF"
date: 2025-06-08 00:00:00 +0000
categories: [CTF, Web, Forensics]
tags: [CTF, Goldrush Gauntlet, Walkthrough, Web Security, Forensics]
---

Welcome to my walkthrough of the Goldrush Gauntlet CTF challenges. In this writeup, I'll be sharing my journey through various security challenges and the techniques used to solve them.

# Forensics Challenges

## Video Valuables

![Challenge Description](/Imgs/CTF/goldrush/forensics/video/challenge.png)

### Initial Investigation

1. The challenge provided two initial files:
   - A text file containing a forum conversation (`copy_of_posts.txt`)
   - A video file (`output.avi`)

<video width="100%" controls>
  <source src="/Imgs/CTF/goldrush/forensics/video/output.avi" type="video/x-msvideo">
  Your browser does not support the video tag.
</video>

2. The forum conversation revealed an interesting exchange:

{% include embed.html url="https://youtu.be/CkmDjbpZu8k" %}

```text
johndeerlover9000:
Let me know what you think about this - i think its pretty sick :-)
~ 6120E 4 lyfe
_____________________________________________________________________________________

farmerboi2008:
Hey man i think you left something visible that you weren't?
~ M6-101 riderz
_____________________________________________________________________________________

johndeerlover9000:
uhhh oops (⊙_⊙;)
~6120E 4 lyfe
```

### The YouTube Trail

3. Here's what the video looked like:

![Video Screenshot](/Imgs/CTF/goldrush/forensics/video/output.png)

4. In the video description, I found an interesting note:
   ```text
   note to self don't leave this in when I upload
   vasvavgrfgbentrtyvgpu
   ```

### Discovering ISG (Infinite Storage Glitch)

5. Research led me to discover that ["Infinite Storage Glitch"](https://github.com/DvorakDwarf/Infinite-Storage-Glitch) is a steganography tool specifically designed for hiding files in YouTube videos. The tool allows embedding any file type into a video that can be uploaded to YouTube. For a detailed guide on using ISG, you can refer to [this excellent tutorial](https://techkamar.medium.com/store-any-file-without-storage-limit-using-youtube-and-this-awesome-tool-f2b224f6e6d).

6. After successfully setting up ISG, I was able to extract a ZIP file from the video.

### Cracking the Archive

7. The extracted ZIP contained two files:

![First Image](/Imgs/CTF/goldrush/forensics/video/1.png)

![Important Image](/Imgs/CTF/goldrush/forensics/video/important.jpg)

8. Initially, I tried to extract potential passwords from `important.jpg` but had no success.

9. Then I remembered the ROT13 encoded text from the video description - `vasvavgrfgbentrtyvgpu`. Using this as the password for the 7z archive worked!

### Flag Obtained

10. Inside the decrypted archive, I found the flag:

![Flag](/Imgs/CTF/goldrush/forensics/video/videoflag.gif)

### Key Lessons
- Always check video descriptions for hidden information
- ROT13 is a common simple encoding method
- Sometimes the same string can be used multiple times (in this case, as both a clue to the tool and the final password)
- Steganography in social media platforms is an emerging technique for hiding data

This challenge provided an excellent introduction to steganography and demonstrated how data can be hidden within seemingly normal social media content.

# Web Challenges

## Hidden Treasures

![Challenge Description](/Imgs/CTF/goldrush/web/challenge.png)

### Initial Reconnaissance

Upon visiting the challenge URL, we're presented with a simple login page:

![Initial Login Page](/Imgs/CTF/goldrush/web/1.png)

### The Hunt Begins

1. First, I explored the site's directories and found a registration option. However, it required an invite code:

![Registration Page Requiring Invite Code](/Imgs/CTF/goldrush/web/6.png)

2. After examining the page source, I discovered an interesting JavaScript file named `inviteapi.min.js`. Its contents were obfuscated:

```javascript
eval(function(p,a,c,k,e,d){e=function(c){return c};if(!''.replace(/^/,String)){
    while(c--)d[c]=k[c]||c;k=[function(e){return d[e]}];e=function(){return'\w+'};c=1;}
    while(c--)if(k[c])p=p.replace(new RegExp('\b'+e(c)+'\b','g'),k[c]);return p;}
('2 0=3(){4"Use base64 to decode this: L2ludml0ZS8xX21vcmUvY29kZQ=="}',5,5,
'generateInvite|function|var|function|return'.split('|'),0,{}));
```

3. The file contained a base64 encoded string. Decoding it revealed a path: `/invite/1_more/code`

4. Visiting this path provided me with the invite code:

![Invite Code Page](/Imgs/CTF/goldrush/web/4.png)

### Digging Deeper

5. After creating an account and logging in, I continued examining the source code and found another JavaScript file: `adminapi.min.js`:

```javascript
eval(function(p,a,c,k,e,d){e=function(c){return c};if(!''.replace(/^/,String)){
    while(c--)d[c]=k[c]||c;k=[function(e){return d[e]}];e=function(){return'\w+'};c=1;}
    while(c--)if(k[c])p=p.replace(new RegExp('\b'+e(c)+'\b','g'),k[c]);return p;}
('2 0=3(){4"Fraq CBFG gb /nqzva-haybpx jvgu obql: { xrl: \"yrgzrva\", hfreanzr: \"<lbhe_hfreanzr>\" }"}',5,5,
'elevate|function|var|function|return'.split('|'),0,{}));
```

6. After decoding, I found that I needed to: `Send POST to /admin-unlock with body: { key: "letmein", username: "<your_username>" }`

### Striking Gold

7. I sent the POST request using curl:

![Curl Request](/Imgs/CTF/goldrush/web/7.png)

8. Finally, logging back in with my credentials (z3r0xk:123) revealed the flag:

![Flag Obtained](/Imgs/CTF/goldrush/web/flag.png)

This challenge demonstrated the importance of thorough source code review and understanding of common web security concepts like authentication bypass techniques. 