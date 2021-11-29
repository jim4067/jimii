---
layout: post
title: The Rust Programming Language
excerpt: An gentle intro to the rust programming language.
date: 2021-10-30
updatedDate: 2021-11-13
tags:
    - post #Forgetting this line is unadvisable
    - rust
---

Ladies and Gentlemen, boys and girls, grab onto your seats because I'm about to take you on the ride of your life. Elon Musk and co. might be trying to sell you the dream of commercial spaceflight, but what I bring you is even better. *Drum roll please*, On this intergalactic voyage we'll explore and get to know the humble beginnings of this gem known as Rust, and what makes it so special that, that colleague of yours won't keep quiet about it. Change can be scary but it doesn't always have to be, so try reading the article with an open mind, and learn Rust.
<!-- more -->

The project began as a personal project of Mozilla employee Graydon Hoare back in 2006 before being open sourced by Mozilla in 2010. Rust was designed to be a safer alternative to C/C++ and enable programmers safely deal with memory and not have to worry about performance taking a hit. According to the the official website *Rust is a language empowering everyone build reliable and efficient software*. 

According to a [blog by Microsoft Security Response Center (MSRC)](https://msrc-blog.microsoft.com/2019/07/16/a-proactive-approach-to-more-secure-code/), since 2004, the majority of vulnerabilities fixed were caused by developers inadvertently inserting memory corruption bugs into their C/C++ code. A whooping 70% of them being in some way related to memory! Now while the latest versions of these languages have tried to address some of these issues with the addition of features like smart pointers, writing C++ code still remains a daunting task that only the chosen few should undertake. In this area, Rust shines as it takes a first class approach to solving memory related issues by introducing a memory model known as ownership where you are guaranteed to never accidentally shoot yourself in the foot when dealing with memory since most of the bugs can be isolated and eliminated during compilation. 

Moreover, is ownership model cost's nothing in terms of resources and perfomance as would other high level languages say those that use the JVM, hence _zero-cost abstractions_. What you don't use, you don't pay for as famously coined by Bjarne Stroustrup. This mean that you can use Rust for performance critical systems, such as distributed systems or bare metal systems. *Reliable* and *efficient* , a mantra you can proudly and confidently live by when using the language. 

The language is relatively new, with version 1 being released in 2015. Understandably the user base is quite small, with a majority being hobbyist programmers. During those six years though, the language has taken the top spot for the most loved language on the [Stack Overflow developer Survely](https://insights.stackoverflow.com/survey/2021#most-loved-dreaded-and-wanted-language-love-dread). A clear demonstration of confidence in the language. This success is largely attributed to the helpful and welcoming community behind the langue. In addition to traditional paperback books there is also a plethora of resources for getting started most of which are build and made available freely by the wider community online.

The ease by which someone can get information to getting started with the language. Not too much to lead to information overload (I'm looking at you JavaScript) or too little that you get frustrated when searching for learning material. Additionally, the inclusion of a [learning page](https://www.rust-lang.org/learn) to guide curious new learners is the icing on the cake. Should you get lost or stuck while learning the language there are also [official](https://discord.com/invite/rust-lang) and [community](https://discord.com/invite/aVESxV8) discord channels where you can ask questions if you get stuck. Not to be forgotten is the [official language forum](https://users.rust-lang.org/). The are also several meetup groups worldwide and several conferences that happen all year round if you fancy IRL events.

Although the user base is small the language has gotten recognition by major corporations, one mentioned earlier being Microsoft who decided to move forward with using rust to alleviate memory related problems they encounter in their code base. Dropbox who completely [rewrote their entire their entire sync engine](https://dropbox.tech/infrastructure/rewriting-the-heart-of-our-sync-engine) in Rust and the latest addition to this ever growing list being Vercel - the company behind Next - For the latest release of Next.js 12, they decided to switch to the [swc JavaScript/TypeScript](https://swc.rs) compiler written in Rust, whose benchmarks blow babel out of the water on both single and mutli-core environments. The list of companies can be found on [this page](https://www.rust-lang.org/production/users) and keeps on growing a testament to language's ability to back up it's mantra of reliability and efficiency.

To close my case for the language, what does the future hold for the language? In the beginning of 2021 the Rust Foundation was created. This non-profit, would be the legal non-profit entity to manage the language ensuring that the Rust train keeps on chugging on ahead.

May the force be with you ✌️ ....
