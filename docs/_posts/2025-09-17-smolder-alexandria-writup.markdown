---
layout: post
title:  "CSAW Quals 2025: Smolder Alexandria Writeup"
date:   2025-09-17 12:30:09 -0400
categories: web
---
`Smolder Alexandria` is a web challenge whose main functionality we can access is a a log look up system. Based on the search, it returns a series of logs corresponding to that search (essentially a grep). We needed to find a way to access `flag.txt` and the data within it to solve the challenge.

**Preliminary Testing**
First, I tested some vault searches and payloads to test for common vulnerabilities. First, I tried to test for SQL injections and Command Injections. I tried `'` but it doesn't return anything quite interesting, but when I try using the character \` in for example, \`whoami\`, I find that the character is blocked. A blacklist can sometimes point to a vulnerability that was found and attempted to be patched, so I focused on finding ways to command inject while bypassing the blacklist.

**Gathering Info**
I tested which characters were allowed and which weren't allowed using a simple Python script. I first ran the website through Burp Suite proxy, tried to send a vault search, and captured the request to retrieve the request information. Once done, I made a simple Python script to try every ASCII character and output the raw response and whether it was blocked or not. I outputted this to an excel sheet for easy viewing.
![Blacklisted EXCEL]({{ '/assets/img/smolder-alexandria-0.png' | relative_url }})

We can see that the following characters are not allowed:
```
"
&
'
;
`
|
```

So, looking through different command injection payloads and ruling out those using blacklisted characters, I decided on using `${payload}`. I confirmed it worked by searching `{echo an}`, and saw the results were the same as we got with just searching up `an`. This meant that whatever was used to search through the vault executed our payload, and searched using the output of the execution (in this case `an`) and returned the results as normal.

***Trying stuff out***
It seemed HTTP requests outside were blocked, and there didn't seem to be a way to get back the output of our execution. Trying to for example do `{cat flag.txt}` would simply search the vault for the value of the flag, and since it didn't return anything, not return any logs in the vault at all.

Me, Txaber, Nash, and Saketh worked on solutions involving ***which*** results were returned. The solution they worked used a script to essentially search using the characters of `flag.txt` one at a time (using `dd`), and by determining which results were returned, they could figure out which character it belonged to. However, not every result signature for every character in the flag was unique, so we ended up with a string that was close to the value of ***flag.txt*** but not quite there.

Meanwhile, I saw the digits (0-9) DID have unique results signatures, so I worked on a solution that was similar, but converted each character in the flag to a three digit equivalent, which I could use to figure out the flag. However, I ran into some problems with some packages I needed not being installed on the system.

What I noticed while working on this solution was that trying to execute a command that didn't exist resulted in one of the logs returned actually containing an error about the specific command not existing. For example, when I tried to execute `somerandomcommand`, a log revealed:
![Running somerandomcommand]({{ '/assets/img/smolder-alexandria-1.png' | relative_url }})

This meant the error stream was leaking through the log! I made a simple solution, I simply called the value of `flag.txt` as a command and got the flag from the output. I ran `${${awk 1 flag.txt}}`, got the flag, and submitted it.
![Getting the flag]({{ '/assets/img/smolder-alexandria-2.png' | relative_url }})

**Conclusion**
It was a bit disappointing seeing how simple the solution was compared to the solutions we were trying to implement. But this was an overall very fun challenge!

**Flag: csawctf{5m0ld3r_bUrN_1mm0l473_R463}**