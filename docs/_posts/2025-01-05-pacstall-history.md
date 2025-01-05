---
layout: post
title: A history of pacstall
subtitle: The history from my perspective
gh-repo: pacstall/pacstall
gh-badge: [star, fork, follow]
tags: [pacstall]
comments: true
mathjax: false
author: Elsie
---

## Very Beginning
My idea of pacstall firstly came out of boredom. I was 14 at the time, and it was during COVID, and I was slowly descending into the depths of Linux. I had been using Linux since I was 8, but the mind of a kid isn't really that interested in the complexities of their computer, but at 14 something clicked, and with my marginal skills in bash that I had aquired, I decided I should make something. I don't exactly remember where my idea for a package manager came from, but it probably stemmed from the fact that it was the most interesting thing my 14 year old brain saw on my system, and it didn't seem "too hard". So I decided to start on it. I was really stupid so I started uploading tarballs to my repo (I had no idea of how git really worked), and started to work on a preliminary system that followed this general structure:

1. Get package tarball from repo.
2. Get individual bash scripts from repo and run them (such as `build.sh`, `install.sh`).
3. Install with (and this part changed very early from `checkinstall(8)` (`577fab6a168aba4fea69451902fc86c48ba039ee`)) porg[^1] (`b0d8c31de3a140fba1265fef51d12bbadebcd335`).

The entire purpose of pacstall was simply to orchestrate building packages as they came from upstream and interface as little as possible with the system, to the point where you could eventually drop in a tarball with instructions and let pacstall handle everything on any system it was on. Essentially I wanted a universal source-based package manager that sat on top of the hosts package manager for its primary dependency management (which the last part isn't actually too far off from what we have today, so that part survived).

Now looking at this retrospectively, it's pretty smart (the concept not the execution), but as a 14 year old, I didn't have a grand vision and sort of did whatever I thought ought to be the "new way forward", and things were very fluid early on; the only major thing guiding me was it had to interface with the host package manager for dependencies (because I wasn't a good programmer I didn't want to make that system myself). Honestly I felt a little bit like a fraud: resting on the laurels of giants (package managers) where all I was doing was making a thin "package manager" on top of theirs. That feeling has still persisted till today and I don't think I've told anyone about that yet until now.

At the end of this whole endeavour, I "supported" a couple different OSs (`a97989b41af4bd8e391c40059ba91eee2e8b68e0`), of which only Ubuntu survived till today.

## Getting smarter
So during the next "generation" of pacstall, I had been learning a lot more about package managers and how they really work, and I had been putting it into use. I decided on the two biggest changes I had done up until that point:

1. Replace build scripts and integrated with a PKGBUILD-esque system (`90b3c75da18bd94798754e7bd3838a9004e5b3d6`).
    * This was to consolidate the complexity of build logic and package logic into one file.
2. Replace porg with GNU Stow[^2] (`f748d3a32989f3bfea605829dd5e9a37ca8f36a1`).
    * This was to "partition" out installed packages from what had been installed straight onto `/` into a place where I could just `rm -rf` it and it was gone from the system.

Then I took a break. I got bored; didn't see much value in it. This is probably where I felt the most fraudulent of all. I felt that if I couldn't actually make something without in my mind "hijacking other package managers", what was the point?

## Coming back
My break lasted a while, and I didn't think of it as a break at the time, I just thought pacstall was "that cool experiment". I don't remember the event that got me reinterested in it, but it was 2021 at this point, and after a *lot* of work and introspection I decided my program was passable enough to share. Originally, I had hoped that pacstall would be used by one or two other people and just spread from there so that I would have to do 0 PR to make it known, but after a year, I decided I couldn't wait for that to happen, so I went to Reddit and posted about my project[^3]. I was really scared that I would get a lot of hate, because internally I still felt like a fraud and my worst fear was putting myself out there for people to see it.

A couple things happened from there in a short amount of time, so when reading these next couple paragraphs, remember that these all happened roughly within a couple weeks and I remember all this happening like a whirlwind:

* The reddit post was received alright. There were some people that didn't like it, but the majority of the people just saw it like "huh neat".

* DistroTube made a video[^4] that took the dissapating hype for pacstall and rebounded it. I remember just breaking down in tears on the porch seeing the notification for my project being taken seriously by someone who I considered "my favorite Youtuber".

* A couple core contributors, namely Wizard[^5], D-Brox[^6], and Astro[^7] joined the project. They were the core of pacstall alongside me, and they are the reason it is what it is today.

* We released 1.3 Amaranth[^8], which marked the first release where other contributors existed besides myself.

## Afterwards
The next year is pretty boring (just updates as normal and slowly extending what we already had), so I will go to RRR. I forgot exactly how I met AJ, but I did, and whole story short, he had a rolling-release Ubuntu distro, which I found kinda cool, so we integrated pacstall into it as an enhancement. About a year after RRR integrated pacstall, we decided to shut it down in favor of a successor, because the entire OS (much like pacstall) was held together by duct tape and holy water, and it wasn't practical to continue to extend it anymore, so we made Rhino Linux, but this time, pacstall wouldn't just be a side piece, it'd be a full on core component. Some time after, Oren[^9] joined, and at first only worked on the RL side, before coming over to pacstall and basically overhauling everything we had and extending pacstall way past my wildest dreams.

---

## Sources
[^1]: https://porg.sourceforge.net/
[^2]: https://www.gnu.org/software/stow/
[^3]: https://www.reddit.com/r/Ubuntu/comments/n4bleb/an_aur_like_system_for_ubuntu/
[^4]: https://www.youtube.com/watch?v=MFbO60gBRLg
[^5]: https://github.com/wizard-28
[^6]: https://github.com/d-brox
[^7]: https://github.com/0oAstro
[^8]: https://github.com/pacstall/pacstall/releases/tag/1.3
[^9]: https://github.com/oklopfer
