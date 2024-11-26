---
title: "#FuckStalkerware pt. 4 - the truth come out: does TheTruthSpy is secure"
date: 2024-02-12
description: "they had like two years to fix this shit, jesus christ"
feature_image: /img/posts/fuckstalkerware-4/cover.jpg
feature_alt: "a glitchy edited very pink collage of various marketing images for TheTruthSpy with their logo diagonally in the center"
tags:
    - "#FuckStalkerware"
    - shady business
    - investigation
    - stalkerware
    - research
    - analysis
    - leak
    - exploit
    - asp.net
    - security
content_warnings:
    - mentions of abuse/controlling behaviour
---

*if you want to know if your phone is/was infected with TheTruthSpy's stalkerware you can use the [lookup tool provided by TechCrunch](https://techcrunch.com/pages/theTruthSpy-investigation/), which has been updated to include all data from this leak as well. to learn about what stalkerware is and why it is bad read the [first entry of this series](/posts/fuckstalkerware-0/).*

just how many times can a company get hacked in exactly the same way until they (hopefully) start giving a shit? the answer turns out to be at least **four times**, which let's jusr say, is quite often. [originally hacked twice in 2022](https://techcrunch.com/2022/02/22/stalkerware-network-spilling-data/) TheTruthSpy (and its parent company 1Byte) may have just broken some sort of world record, getting hacked again by [ByteMeCrew](https://t.me/ByteMeCrew) and [SiegedSec](https://t.me/SiegedSecurity) in the exact same way and then *again* by me in about four different ways.

when members of the two hacking groups looked into TruthSpy last december while searching for stalkerware to hack, they independently stumbled upon the same [IDOR](https://en.wikipedia.org/wiki/Insecure_direct_object_reference) <small>(help out wikipedia and anyone trying to learn more by expanding this article)</small> vulnerability originally reported on by TechCrunch ([CVE](https://nvd.nist.gov/vuln/detail/CVE-2022-0732)), which evidently had not been fixed yet. this easily exploited type of vulnerability appears when data is retrieved without verifying that the client has the right to do so. a query parameter denoting a user ID (`like https://example.com/users/?id=123`), for example, allows for simple enumeration and scraping of data that is typically difficult to access otherwise. since this vulnerability in TruthSpy STILL isn't fixed at the time of writing i am unfortunately unable to publicly talk about where and how exactly this vulnerability was exploited as i usually would.

upon exclusively receiving the data from ByteMeCrew to report on it i decided to contact [Zack Whittaker](https://techcrunch.com/author/zack-whittaker/) (who wrote the aforementioned TechCrunch article) to collaborate on the research and reporting (shoutout to Zack for being one of the very few journalists writing about stalkerware btw, much love). he quickly got to reaching out to the server hosters and payment providers used by TruthSpy to inform them of the TOS violations. the resulting game of whack-a-mole ended with them switching to Moldovan hosting company [AlexHost](https://alexhost.com), who has yet to respond to any requests, and moving checkouts to p2p payments (presumably via crypto) via their support portal (after 1Byte initially scrambled to use the checkout experiences of their non-stalkerware projects for TruthSpy as well).

with most of the in-depth reporting on TruthSpy's background already having been done by Zack two years ago, i decided to focus my research on trying to uncover more of the inside workings of 1Byte, their various non-stalkerware ventures (primarily online language learning apps), their CEO Van "Vardy" Thieu and the full breadth of their disregard for cybersecurity 101.

### 1Byte, i am inside you

> take me down to [IDOR city](/img/posts/fuckstalkerware-4/what-is-this-mp4-city.mp4) where the bugs are dumb and the site is shitty

one of the first things i wondered was whether i could find an employee with TheTruthSpy on their personal device for testing and indeed, a search for "1Byte" in the leaked data reveals a device that has two contacts containing "1Byte" in their name as well as various contacts for test devices, most of which included the device model in the name.

![a heavily redacted screenshot of visual studio code, showing a csv file as a search result for the term "1Byte" with various contacts related to test devices and 1Byte employees](/img/posts/fuckstalkerware-4/1byte-contacts.jpg)

one of the two employees found there publicly identifies himself as a database engineer working at 1Byte on linkedin. the other one is harder to track down, though their phone number is linked to a [vk](https://en.wikipedia.org/wiki/VK_(service)) account active all the way back in 2014, advertising a predecessor to what has now become the TruthSpy network of apps. we can't gather much more info from this test device though, as only contacts were actually ever synced.

sticking to general 1Byte infrastructure i scanned for subdomains on the 1Byte corporate domain and just like TechCrunch found partial and public Jexpa Framework (the common framework/backend used for all 1Byte apps) documentation. yep, they fucked up again. what i found this time was even more interesting though: two different supposedly internal portals for employees. the first of the two was a [gitbook](https://www.gitbook.com/) documenting most of their employee onboarding process and company structure, conveniently including an organizational diagram of the company, among various other bits of interesting corporate data. 

![an organizational diagram of the 1Byte company mentioning various management people and departments in vietnamese](/img/posts/fuckstalkerware-4/1byte-orgchart.jpg)

and yet the second one, a [freshdesk](https://www.freshworks.com/freshdesk/) helpdesk for employees, would prove to be even more useful for gaining an inside look at 1Byte. contained within the knowledge base is information on the per-department software stack, salary information, detailed employment benefit terms, a guide on buying botted facebook likes for their marketing team as well as a link to a publicly viewable time tracking google sheet. over the various sheets i was able to see all the logged time in 2023 for all 1Byte employees, their full names, whether they work full or part time and their roles at the company.

![a partially redacted screenshot of a vietnamese google sheet showing time tracking info of all 13 1Byte vietnam employees in september 2023](/img/posts/fuckstalkerware-4/1byte-timesheet.jpg)

now with a basic understanding of the company internals it was time to start exploring their various products to try and find some more interesting data. monitoring the sites network traffic using firefox developer tools i signed up for one of their websites, a daily english pronounciation training app. clicking around on the site and using various features i got a bit of an idea of the shared Jexpa backend. making some bold assumptions about the security of the API i almost immediately found multiple IDOR vulnerabilities, letting me grab all user data and even analytics tracking data without any authentication of any kind. matching this user data (about 35 thousand users across all applications) to the employee names we found earlier gave me a list of personal email addresses and phone numbers for some of them, which i used to reach out for comments, very curious to see if anyone would be willing to speak to me about the company. at the time of writing none of the three current and former employees i reached out to have given any response, this article will be updated if they do.

because it wouldn't be a [#FuckStalkerware](/posts/tagged/fuckstalkerware/) post anymore without a bit of webshell action, i tested the profile picture update endpoint for arbitrary file uploads and as is standard by now it sure allowed for it, once again without authentication of any kind. the problem i began facing then though was having to deal with [ASP.NET](https://en.wikipedia.org/wiki/ASP.NET) for the arbitrary code execution, since their infrastructure all runs on windows. since i want to avoid ever having to deal with a windows cmd shell and am lazy as shit i spent quite some time finding a cozy enough advanced ASP webshell that also doesnt immediately break when i try to use it. thankfully i eventually found something that would work in my favorite [chinese webshell collection on github](https://github.com/tennc/webshell) (look, doing skid shit is fun and easy, we all do it sometimes, especially when working with windows, and i have already written more than enough single-use tooling for this research). while this shell generally worked really well for what i wanted to do i still somehow managed to crash the Jexpa CDN .NET instance, semi-breaking their apps for almost a day, with the person who eventually fixed that not even noticing my intrusion. awesome ^-^

![screenshot of the r00ts.asp webshell running on the jframework webserver. it is in green on black design and most ui text is in chinese](/img/posts/fuckstalkerware-4/r00ts.jpg)

with the shell planted on the system i did my usual prowl through the file system, copying anything i was interested in (backups, webserver .NET binaries, etc) to the public cdn directory so i could very quickly download them using [aria2c](https://aria2.github.io/). digging through this data i didn't really find much of note that i hadn't already grabbed via IDOR, but it was still an interesting insight into their infrastructure and another testament to their bad security.

finally having a separate test phone again to run shady software on in an isolated context, i set out to also actually look over the TruthSpy attack surface as well. a decompilation i did of the client earlier showed that the app contacts its backend completely over plain HTTP, which is a massive privacy and security issue due to how easy it makes the interception of all traffic when on the same network, like on public wifi. however, it also makes my analysis super easy :3! i set up [ZAP](http://zaproxy.org/) to intercept all traffic from the phone and installed the client. it was apparent that the device api still has no authentication beyond sending the device id with all requests. 

trying to get a potential way to drop another webshell i played around with the photo history and remote photo features, "remotely" taking a photo of myself without touching the phone at all. it definitely felt kind of scary, even after all the research ive already done into stalkerware by now. after some futile tries to use the endpoint to upload arbitrary files i gave up, but as Zack has previously [reported on data leaked from the TruthSpy server](https://techcrunch.com/2023/07/20/thetruthspy-stalkerware-forged-passports-millions/) already i wasn't too bummed. looking through all the other endpoints called by the client i ended up finding the final IDOR vuln to complete the data puzzle i'd been assembling, a way to get the abuser account email addresses from the device ids i already had from the ByteMe leak. however i only managed to grab emails for about half of all affected devices, as the device api is seemingly just broken for a large number of (older) devices, returning internal error messages (mostly null pointers and array out of index errors) rather than the requested data.

{% figure { src: '/img/posts/fuckstalkerware-4/tts-selfie.jpg', alt: 'a low resolution selfie of me covering my face with one hand doign the peace sign and holding my other hand up in the air. i am wearing a merch piece from the musician femtanyl, a long sleeve and headphones', caption: 'the hands free selfie i remotely took using TruthSpy' } %}

### digging your own grave 101

before publishing this article i reached out to Van "Vardy" Thieu to give his company time to fix all of the discussed security issues and give a chance to comment ahead of release. after somewhat pressing him on the urgency of these issues multiple times he committed to getting his team to fix at least some of the issues "after 8 hours". with nothing having changed two days later i gave him another heads up, to which Thieu gave a longer and clearly pretty annoyed response, with him among other things stating that they "do not care of it too much" about the reported security issues, blaming it on their small team size and an alleged lack of funds, saying it is "a low priority".

> we dont have enough time to explain [to] you. so you can write what you want. <span>Van "Vardy" Thieu, CEO of 1Byte</span>

regarding the publicly linked and accessible google sheets with employee information he claims that "it is internal info and it is not allowed to be public by you. [...] it looks like you are a hacker and want to expose info without permission", refusing to fix even that. he also further repeats his past claims of not being involved with TruthSpy (which according to Thieu has "another name (not the name you wrote)"), claiming it is an outsourcing project for a customer done "7,8 years ago" that they have no control over anymore. this is verifiably wrong: the current server infrastructure for TruthSpy references the name of another business publicly associated with Thieu and checkout infrastructure of various 1Byte projects is being used for them, so even if it were a project they operate for a third party it still is fully under their control.

### conclusion

the main thing this long investigation has shown is that not only is the stalkerware platform by 1Byte horribly insecure, but also all their other ventures, such as the educational tools under the VardyTest brand are no different. expecting this company to ever change or improve seems foolish looking at the track record they have paired with Thieu's statements, but the only way forward i see for them is to completely shut down operations and disband to show they at least somewhat care about user safety, even if it's too little too late. more broadly this once again shows that most companies who run stalkerware operations primarily do so as a quick and easy cash grab, but dealing with the various legal, ethical, and moral risks and challenges only really makes sense if all other ventures are horribly failing. 

while i am not one to believe in regulation as a proper solution to combat problems such as stalkerware i would hope to at least see some sort of legal action against a company like 1Byte, who has been running a [massive fraud operation](https://techcrunch.com/2023/07/20/thetruthspy-stalkerware-forged-passports-millions/) for years while enabling domestic abuse at a massive scale, all with a provable complete disregard for user security, even if it's just symbolic.

*if you have any data, insider info, vulnerabilities or any other tips related to stalkerware (or in general) you can securely reach out either to [me](/contact) or [Zack](https://techcrunch.com/author/zack-whittaker/).*