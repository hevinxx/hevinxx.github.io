---
layout: post
title: "Just Search for It on a Portal: A Security Vulnerability I Found in Civil Defense Training"
date: 2026-05-19 09:00:10 +0900
categories: 에세이
tags: 보안, 피싱, UX
description: A government notice told me to take civil defense training online, but instead of giving me a URL, it just said to search for it on a portal site. The same government that is famously strict about security somehow didn't seem to care whether users actually arrived at the real site. Here are some thoughts that came out of that experience.
use_math: false
---
In Korea, every adult male is required to complete annual civil defense training for several years after their military service. I recently received the notice for mine. As a third-year-and-up member, I'm allowed to skip the in-person session and take the training online instead. But there was one line on the notice that left me uneasy.

> How to access: Search for "Smart Civil Defense Training" on a portal site.

From a security standpoint, that single line is dangerous. You might call it an overreaction, but when you consider how many millions of people across the country receive this same notice, even a small risk shouldn't be brushed aside.

The notice doesn't specify which portal site to use. But on most Korean portals, the top of the search results is filled with ads. Anyone with money can buy those slots. That means it's perfectly possible to build a fake site that looks like the real one and float it to the top of the results for "Smart Civil Defense Training." Someone who searches in good faith, just following the notice, would very likely click the first plausible result, trusting that it's official. They'd be walking straight into a hacker's trap, just by doing exactly what the notice told them to do.

The other visual cue we use to trust a site, the domain, wasn't reassuring either. For a government site, you'd hope to see `.go.kr`, or at least the `.or.kr` that public institutions often use. But the site the notice pointed to was just `.kr`. A `.kr` domain can be registered by anyone, including individuals, as long as they meet some basic criteria.

In the end, I called the civil defense contact number printed on the notice and asked whether the site was really run by the government. The person on the line sounded like she was taking that kind of question for the first time. I had to spend a while explaining what I was even worried about. Once I got my confirmation and logged in, the site asked me for my name and phone number. If someone with bad intent had set up a phishing site on a similar domain, bought ads for it, and asked not just for name and phone but for a resident registration number too, how much personal data would have leaked from this one notice?

I later looked up why the URL wasn't a `.go.kr`, and there's a reason. The Ministry of the Interior and Safety's online civil defense training isn't run by the government directly. It's outsourced to private vendors. Different local governments can contract with different vendors, which is why several sites, such as "Smart Civil Defense," "Digital Civil Defense," and "Korea Public Education Institute," exist at the same time. The "Smart Civil Defense Training" I was directed to is one of these. The government isn't being lazy about its domain; structurally, it can't use `.go.kr` for something it doesn't operate.

That doesn't let them off the hook, though. From a user's perspective, that outsourcing arrangement is invisible. What they see is a notice from the government pointing them to something with a government-sounding name. Asking users to mentally untangle a contract structure to figure out whether a site is legit is, frankly, strange.

What made this experience especially bitter is that, in other contexts, the Korean government has been famously strict about "security." Anyone who has tried to do administrative work online has had to install a pile of security programs. It's gotten better lately, but only a few years ago you'd be installing keyboard security software, anti-keyloggers, antivirus tools, and certificate managers, most of which you didn't recognize and couldn't really evaluate.

On one hand, you're made to install all of that in the name of security. On the other hand, you're told to just search for the site on a portal. The most basic question, "Is this actually the right site?" is left entirely up to the user. It's like fitting a car with five different locks and then leaving the key on the curb for anyone to pick up.

So what's the fix? "Just put the URL on the notice" sounds obvious, but it isn't great either. A hacker could send out fake notices with fake URLs, and people would follow them in good faith. That's smishing. Putting a URL on a printed notice nudges people to click without scrutiny, but they have no real way to verify that the URL they're seeing matches the one the government actually printed. That's probably exactly why the original notice told people to search a portal instead.

The better answer is to anchor everything to a single, well-known entry point for government services. For example, tell people to go to Gov24 (`plus.gov.kr`) and find "Smart Civil Defense Training" from there. If there's one entry point simple enough for users to memorize and trust, and they navigate to outsourced sites by following internal links from there, they don't have to evaluate the trustworthiness of a new domain every time. This doesn't require any expensive or cutting-edge technology. You just have to decide, once, where the first step is going to be.

As a developer, I see versions of this all the time. We obsess over how to transmit data securely, which encryption to use, and which auth method is the most appropriate. We compare libraries and weigh algorithms. But we sometimes pay much less attention to the very first step: making sure users actually arrive at our service in the first place. And it's precisely that first step that can quietly nullify every lock we've carefully installed downstream. One bad line of guidance, and the user is at a fake site, where none of our defenses buy us anything.

It gets worse on mobile. Users increasingly don't look at URLs at all. In-app browsers often hide the address bar or omit it entirely. When the environment doesn't let users verify the domain, and the notice only gives them a search term, getting lost is the path of least resistance.

Long before expensive, complex security solutions, a single well-written line of guidance can save a lot.
