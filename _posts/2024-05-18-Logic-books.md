---
title: "Pay 0$, Get ∞ books!"
data: 2024-05-18 00:00:00 +0000
categories: [Web, Gems]
tags: [Logic, web]
---

Hey there, hackers!

![hi](https://miro.medium.com/v2/resize:fit:640/format:webp/1*oMMywOPzIIyUFV1VZhS7pQ.gif)

In this article, I’m going to spill the beans on my latest pentesting adventure with my friend [Hx0_0h4nf1](https://medium.com/@Hx0_0h4nf1) on a private book-selling website, x.com. I stumbled upon a Business logic bug that lets me buy any number of books for the grand total of $0. Ready to dive into the chaos? Let’s go!

![ready](https://miro.medium.com/v2/resize:fit:640/format:webp/1*FMh2M8m7LRrQVuy2F3BINA.gif)

As usual, I started by checking for any of the OWASP Top 10 bugs like CSRF, SQLi, and others, but I didn’t find anything. So, I decided to look for logic bugs because they tend to pop up everywhere. I thought, “Let’s buy some books and see if there’s an IDOR or something else.” But before that, I took a closer look at the parameters, and that’s when I struck gold! 

![req](https://miro.medium.com/v2/resize:fit:720/format:webp/1*TEK2IM_PKXJSR1Hr9ximFw.png)

When I played around with the parameters ```items[...][qty]=1&items[...][price]=250&items[...][amount]=250```, it didn’t work, and the price didn't change. <br>

![messi](https://miro.medium.com/v2/resize:fit:640/format:webp/1*4oXXCIYFjt5OyrHT1iEWdA.gif)

I didn’t give up and decided to think of a different technique. First, I reviewed my previous notes on similar scenarios. After that, I decided to search for hidden parameters that the developer might have thought were safe simply because they were hidden.

![req3](https://miro.medium.com/v2/resize:fit:720/format:webp/1*bax-pdB2ADqPgHCtFnRong.png)

I found 11 hidden parameters on the page that requests the order, but I had already played with all of them except one shipping_fees. This parameter added shipping fees to the total price. What if I play around with it? <br>

And boom! It happened — I found a logic bug that allows me to buy any number of books for $0. <br>

![gif](https://miro.medium.com/v2/resize:fit:640/format:webp/1*-bkIKXZ9O3fGW5w88WPs1A.gif)

**And finally** 

![messi deal](https://miro.medium.com/v2/resize:fit:440/format:webp/1*sr3Yw7sEtUACWyYsLgQFGw.gif)