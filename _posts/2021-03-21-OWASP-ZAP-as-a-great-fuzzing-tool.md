---
title: OWASP-ZAP is it a great alternative for Burp-Suite Intruder?
date: 2020-03-21 06:46:00 +0100
categories: [Web Bug Bounty, Tools]
tags: [fuzzing, owasp, zaproxy]
lang: en
---

## The tools used for fuzzing web-forms / inputs
- ffuf
- wfuzz
- hydra

## One of the biggest problems with fuzzing...
Is that for a beginner - it's not that easy to predefine input fields to fuzz. This often results in constructing a massive command like this:  

```sudo hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.10.43 http-post-form "/department/login.php:username=admin&password=^PASS^:Invalid Password!"```

This kind of fuzzing is not intuitive **for a beginner** (though great for automation)

## Two alternatives come into play:
1. Burp Suite Intruder
1. OWASP ZAProxy

You're probably familiar with the first one - Burp Suite Intruder. It provides an easy way of selecting inputs to fuzz just like on this picture:

![image](https://portswigger.net/burp/documentation/desktop/images/intruder-enumerating-positions-940.png)  

The biggest con of this is that it rate-limits all the fuzzing to the slowest way possible - **if you're not using Professional Edition**  

## A great alternative
Hold your seats though, because there's a great alternative for Burp Suite Intruder - OWASP ZAProxy Fuzzer.  
Which essentially has the same features as the Burp Suite Intruder. But because It's **open-source**, **it is also free**

## Let's see it in actions 
- We'll be testing it on PortSwigger Authentication Academy (Yes, I know...)
- **Do it with me, please :)**

### Lab: Username enumeration via different responses

You can find this lab [HERE](https://portswigger.net/web-security/authentication/password-based/lab-username-enumeration-via-different-responses)

![test](https://imgur.com/68MSbfB.jpg)

We also get the lists that we can use to brute-force these forms:

![lists](https://imgur.com/vFcOttN.jpg)

Let's access the lab, this won't be a walkthrough focused blog post, it'll be mostly about using zaproxy fuzzer. 


Let's open zaproxy, then insert lab's url into URL to explore field and let's start exploring the app on the browser!

![zaproxy-ui](https://imgur.com/DIdjAAH.jpg)

And go to login page, set a breakpoint just like in 1st step and do other steps also ;p.

![login](https://imgur.com/JBhp9uI.jpg)

Then you can see that in the zaproxy ui you have a request exactly on the POST form request  

Right-click on the request, and select the option ```Fuzz```, then this window will appear
![request](https://imgur.com/Y2rs8Uz.jpg)  

If you're familiar with Burp Suite Intruder then you'll probably get comfortable to this interface.  

You're able to add fuzzing placeholders just like in Burp, and add payloads, which soon you'll see are configured exactly as in Burp.  

Highlight the username **value** text (test). When you click 'Add' the new Window to configure Payload/Wordlist will appear. You can tweak with that on your own, there're many interesting options!  

For now though I'll just add wordlist that is given to us by Portswigger Academy.

![payloads](https://imgur.com/yz3wSFS.jpg)

Look like we're done - that's it :) Let's Start the Fuzzer!

After the fuzzing has completed, we can analyse the results.  
As you can see, most of the invalid ones share the same HTTP Code, Size of the Response Body and Size of the Response Header.  

![results](https://imgur.com/BvsjIWv.jpg)

However, we see one has code 302 Found. This appears to be the correct login credentials. I can guarantee, that this process in free version of Burp would take up to 1.5h or even more, when Burp has done that in less than 15 minutes

### Lab: Username enumeration via subtly different responses

You can find this lab [HERE](https://portswigger.net/web-security/authentication/password-based/lab-username-enumeration-via-subtly-different-responses)

Let's repeat the same process again

Fortunately for us, the same process applies and works in zaproxy:

![results2](https://imgur.com/qmMrOFW.jpg)

But it seems like this challenge was not designed in this way. The way it is intended it to grep for a message "Invalid password" and then filter fuzzer's output. But... the problem is that OWASP ZAP doesn't provide this feature in it's GUI version.

Instead, it it support in the [scripting engine](https://github.com/zaproxy/community-scripts/tree/master/httpfuzzerprocessor)




###  Lab: Username enumeration via response timing NOT WORKING ONE :/ 

You can find this lab [HERE](https://portswigger.net/web-security/authentication/password-based/lab-username-enumeration-via-response-timing)

Let's repeat the process again. 

We know (From the hint) that this lab introduces protection against brute-forcing with blocking IP-Addresses, but that it is easy to bypass

This can be bypassed by adding the header 

```X-Forwarded-For: 192.168.1.0```

and fuzzing this ip address, making it impossible for the server (in this scenario) to detect If we're in the same location.

To do this we need to right-click on the request -> Open/Resend with Request Editor -> And then add the header

Then from the Open/Resend Window we right-click, and then choose Fuzz option again.

Now We need to fuzz the ```X-Forwarded-For``` header. Do the same thing as with normal input values - select the last **octet** -> click Add and edit the fields accordingly:

![numberzz](https://imgur.com/o5Nfk4p.jpg)

Then, add only one fuzzing parameter (highlight only one value), and add the payload for username

The final Fuzzer window should look like this:

![window](https://imgur.com/TvfeSes.jpg)

Let's start the fuzzer! 

It seems though that the fuzzer is not working as We expected, because It iterates the ip-address only after all possible usernames were fuzzed already - which results in blocking our ip before even 5 attempts were made.


## Cons of ZAProxy Fuzzer
So we can see many cons with, some of them are:
- Lack of Payload options
- Many features are only accessible through the [community scripts](https://github.com/zaproxy/community-scripts) of ZAProxy

However if someone would dwell deep into ZAP scripting, then it could become a great alternative, that would replace Burp Suite Community Edition.

## For beginners stuff, I think it suits my needs :)