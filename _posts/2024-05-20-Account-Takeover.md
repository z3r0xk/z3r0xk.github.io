---
title: "Your Account is Now Mine!"
data: 2024-05-20 00:00:00 +0000
categories: [Web, Gems]
tags: [Account Takeover, web]
---

![Mine](https://miro.medium.com/v2/resize:fit:640/format:webp/1*2ygmQdER3eGgpaNjRGHa_w.gif)

Recently, I discovered a zero-click account takeover vulnerability in the password reset function of a well-known food brand’s website, let it hamm.com. In this write-up, I’ll explain how this flaw allows attackers to hijack user accounts without any interaction, ready!

![goo](https://miro.medium.com/v2/resize:fit:534/format:webp/1*wDYkZjpQ4BvMkh9HmA_ZAQ.gif)

At first, I tried some known attacks, but unfortunately, I didn’t find anything. I decided to test the password reset functionality because it can often lead to valuable exploits. I created an account for the attacker and an account for the victim then, I decided to examine the request in more detail. I went to the HTTP history tab in Burp Suite and found something unusual that might lead to a potential exploit.

![req1](https://miro.medium.com/v2/resize:fit:720/format:webp/1*QIMfh4x5GbA_wTXhBPtRBQ.png)

In my recent investigation into the security of hamm.com’s password reset functionality, I uncovered critical details by examining the HTTP requests. This analysis revealed three key points:
1. the backend uses the PUT method to edit user passwords,

2. it relies on the Authorization header for user verification, and

3. the passwords appear to be hashed into a 64-bit string.
   
I will experiment with each option to see if I can uncover any vulnerabilities.

![do](https://miro.medium.com/v2/resize:fit:640/format:webp/1*DxT5nsl3yZitySIQtiz7Lw.gif)

First, I experimented with the PUT method but didn’t find anything significant. I then shifted my focus to manipulating the Authorization Header. In my latest findings, I learned some effective tricks, including removing its value and setting it to null, because this approach had worked successfully in the past.

![req2](https://miro.medium.com/v2/resize:fit:720/format:webp/1*ElIKjeovW7BkK7_Sfe0T6w.png)

After changing the value to null, it worked, and the request still returned a 200 OK status. At this point, I thought I had found a lucrative vulnerability. Unfortunately, there was bad news: changing the password to plain text did not work, and the password remained unchanged.

![bad time](https://miro.medium.com/v2/resize:fit:492/format:webp/1*ujx0BvV0sZgI91N6sOGylg.gif)

After some thought, I had an idea: I decided to use the password displayed in the reset password functionality for my attack. Let’s proceed with this approach.

![req3](https://miro.medium.com/v2/resize:fit:720/format:webp/1*TZsZHCXsK_gMlHhFDl5Dbg.png)

![req4](https://miro.medium.com/v2/resize:fit:720/format:webp/1*a4DgteJudBEzDQ2NJbh5fw.png)

Attacker account: (0…011:Attack3rX@P4ssw0rd)
Victim account: (0…737:VictimX@P4ssw0rd)
Hash(Attack3rX@P4ssw0rd) = 2a5d03c3f9b95d2ac057944e0f3502305903898eb7a18dcd3898de620c171906
Hash(VictimX@P4ssw0rd) = 68a32c169d65943b82e0811c96022a7b8092d58e1616cfb96f036e0370204e08
<br>

I will change the victim’s password to ```Attack3rX@P4ssw0rd```.

![req5](https://miro.medium.com/v2/resize:fit:720/format:webp/1*MApiYqQM4OwakIfh1sxKIA.png)

And **boom**, I’m in.

![in](https://miro.medium.com/v2/resize:fit:720/format:webp/1*ipkL0s0s7fOISKMhIhw3Sg.png)

**And finally,**

![messi](https://miro.medium.com/v2/resize:fit:440/format:webp/1*sr3Yw7sEtUACWyYsLgQFGw.gif)
