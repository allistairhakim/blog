---
layout: post
title:  "CSAW Quals 2025: Smolder Alexandria Writeup"
date:   2025-09-17 12:30:09 -0400
categories: web
---
<!-- Per-post responsive image rules: images inside .post-content will scale to container and be centered -->
<style>
/* limit scope to this post's content */
.post-content img.responsive {
  display: block;          /* put the image on its own line so it can be centered */
  margin: 1rem auto;       /* center horizontally and give a little vertical breathing room */
  max-width: 100%;         /* never exceed the width of the container */
  height: auto;            /* keep aspect ratio */
}

/* if you prefer images to sit inline with text, replace `display: block` with `display: inline-block` */
/* .post-content img.responsive-inline { display: inline-block; vertical-align: middle; margin: 0.5rem 0; } */
</style>

<div class="post-content">

<img class="responsive" alt="Picture of challenge" src="{{ '/assets/img/smolder-alexandria-3.png' | relative_url }}" />
# Introduction
`Smolder Alexandria` is a web challenge whose main accessible functionality is a log look-up system (vault). Based on the search, it returns a series of logs corresponding to that search (essentially like `grep`). We needed to find a way to access `flag.txt` and the data within it to solve the challenge.

# Preliminary Testing
First, I tested some vault searches and payloads for common vulnerabilities. I first tested for SQL injection and command injection. I tried some SQL payloads but they didn't return anything particularly interesting, but when I tried using the character `` ` `` in, for example, `` `whoami` ``, I found that the character was blocked. A blacklist can sometimes indicate a vulnerability that was discovered and partially patched, so I focused on finding ways to perform command injection while bypassing the blacklist.

# Gathering Info
I tested which characters were allowed and which weren't using a simple Python script. I first ran the website through the Burp Suite proxy, sent a vault search, and captured the request to learn how to make the request myself (you could also, more simply, open the network tab in Chrome). Once done, I wrote a Python script that tried searching for every ASCII character and output the raw response and whether it was blocked. I exported this to an Excel sheet for easy viewing.

We noted that the following characters were not allowed, everything else was:
```
"
&
'
;
`
|
```
&nbsp;
So, looking through different command injection payloads and ruling out those using blacklisted characters, I decided on using `${payload}`. I confirmed it worked by searching `{echo an}`, and saw the results were the same as we got with just searching `an`. This meant the search mechanism executed our payload, used the output of that execution (in this case `an`) as the search term, and returned the results as normal.

# Trying stuff out
It seemed outbound HTTP requests were blocked, and there didn't appear to be a way to retrieve the output of our execution. For example, running `{cat flag.txt}` would simply search the vault for the flag's value; because that returned nothing, no logs were returned.

Txaber, Nash, Saketh, and I worked on solutions involving ***which*** results were returned. The solution they worked on used a script to essentially search using the characters of `flag.txt` one at a time (using `dd`); by determining which results were returned, they could identify which character it belonged to. However, not every result signature was unique, so we ended up with a string that was close to the value of ***flag.txt*** but not exact.

Meanwhile, I observed that the digits (0-9) did have unique result signatures, so I worked on a similar solution that converted each character in the flag to a three-digit equivalent which I could use to determine the flag. However, I ran into problems because some required packages were not installed on the system.

While working on this solution, I noticed that attempting to execute a non-existent command caused one of the returned logs to contain an error about that command. For example, when I tried to execute `somerandomcommand`, a log revealed:
<img class="responsive" alt="Running somerandomcommand" src="{{ '/assets/img/smolder-alexandria-1.png' | relative_url }}" />

This meant the error stream was leaking into the logs! I implemented a simple solution: I executed the contents of `flag.txt` as a command and obtained the flag from the output. I ran `${${awk 1 flag.txt}}`, retrieved the flag, and submitted it.
<img class="responsive" alt="Getting the flag" src="{{ '/assets/img/smolder-alexandria-2.png' | relative_url }}" />

# Conclusion
It was a bit disappointing to see how simple the solution was compared to the approaches we were attempting, but overall it was a very fun challenge!

**Flag: csawctf{5m0ld3r_bUrN_1mm0l473_R463}**

</div>
