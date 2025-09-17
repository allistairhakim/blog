---

layout: post
title:  "CSAW Quals 2025: Smolder Alexandria Writeup"
date:   2025-09-17 12:30:09 -0400
categories: web
---------------

`Smolder Alexandria` is a web challenge whose main functionality is a log lookup system (the "vault"). Based on a search, it returns a series of logs corresponding to that query (essentially a grep). We needed to find a way to access `flag.txt` and read the data inside it to solve the challenge.

# Preliminary Testing

First, I tried some vault searches and payloads to look for common vulnerabilities. I tested for SQL injection and command injection. Some SQL payloads returned nothing interesting, but when I tried using the backtick character — for example, `` `whoami` `` — I found that it was blocked. A blacklist can sometimes indicate that a vulnerability was discovered and then partially patched, so I focused on finding ways to perform command injection while bypassing the blacklist.

# Gathering Info

I tested which characters were allowed and which were blocked using a simple Python script. I ran the site through Burp Suite, sent a vault search, and captured the request to learn how to reproduce it myself (you could also just open the Network tab in Chrome). Then I made a small Python script that tried searching with every ASCII character and recorded the raw response and whether the character was blocked. I exported the results to a spreadsheet for easier viewing.

We noted that the following characters were not allowed; everything else was:

```
"
&
'
;
`
|
```

Looking through command-injection payloads and ruling out those that used blacklisted characters, I settled on `${payload}` as a usable pattern. I confirmed it by searching `${echo an}` and saw results identical to searching for `an`. This meant the vault executed our payload and then searched using the payload’s output (in this case `an`), returning results as normal.

# Trying stuff out

External HTTP requests appeared to be blocked, and there didn’t seem to be a way to directly capture the output of our commands. For example, running `${cat flag.txt}` would simply cause the vault to search for the flag’s value; if nothing matched, the vault returned no logs.

Txaber, Nash, Saketh, and I worked on solutions that depended on which results were returned. Their approach used a script to search using the characters of `flag.txt` one at a time (using `dd`), and by observing which results were returned they could infer each character. However, not every character produced a unique result signature, so we ended up with a string that was close to the true value of `flag.txt` but not exact.

I noticed that the digits (0–9) produced unique result signatures, so I tried a similar approach that converted each character of the flag into a three-digit equivalent to identify the flag that way. I ran into problems because some required packages weren’t installed on the system.

While experimenting, I discovered something useful: trying to execute a non-existent command caused one of the returned logs to include an error message about the missing command. For example, when I ran `somerandomcommand`, a log revealed an error message showing the command wasn’t found — the stderr was leaking into the logs.
!\[Running somerandomcommand]\({{ '/assets/img/smolder-alexandria-1.png' | relative\_url }})

That meant the error stream was leaking through the logs. I used that to my advantage: I executed the contents of `flag.txt` as a command and read the error output. I ran `${${awk 1 flag.txt}}`, which caused the vault to execute the file contents and print an error revealing the flag. I retrieved the flag and submitted it.
!\[Getting the flag]\({{ '/assets/img/smolder-alexandria-2.png' | relative\_url }})

# Conclusion

It was a bit disappointing how simple the final solution was compared to the approaches we were trying to implement, but overall this was a very fun challenge!

**Flag: csawctf{5m0ld3r\_bUrN\_1mm0l473\_R463}**
