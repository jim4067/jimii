+++
title = "My EthIndia Fellowship Experience"
date = 2023-04-08
category = "Prog"

[taxonomies]
tags = ["EthIndia"]
+++

![](/eif3/eif3cover.png)

It was the best of times, having the chance to finally get back to working in the Ethereum ecosystem and the worst of times having to sit for my exams during the same period.

It's not everyday that lightning strikes twice but that's what happened when I received the email that I had been accepted into the [EthIndia fellowship 3.0](https://twitter.com/jimii_47/status/1620293669638270978?s=20). I hadn't had the time to recover from the high of participating in the MLH fellowship that fall and yet here I was getting on the saddle to accept yet another fellowship.

<sub style="color:#1ccaac">This blog was generated with the help of chatGPT.</sub>

<!-- more -->

# Table of contents

1. [Introduction](#introduction)
2. [Pre-fellowship Prep](#pre-fellowship)
3. [Fellowship Experience](#fellowship-exp)
    1. [learning phase](#learning-phase)
    2. [build phase](#build-phase)
4. [Conclusion](#conclusion)

<!--
## Some paragraph <a name="paragraph1"></a>
The first paragraph text

### Sub paragraph <a name="subparagraph1"></a>
This is a sub paragraph, formatted in heading 3 style

## Another paragraph <a name="paragraph2"></a>
The second paragraph text -->

</br>

## Introduction <a name="introduction"></a>

As per the [website](https://eif3.devfolio.co/),the EthIndia fellowship is an 8-week mentor-led program with 2 parallel tracks, open to all individual. The two tracks consists of

-   **Track Wei** _which I was part of_, aptly named after the smallest Ethereum units, aims to familiarize developers to learn more about web3 with the help of training and mentoring.
-   **Track Gwei** a track for intermediate web3 developers to focus solely on building tools and products in the web3 space.

There were three main reasons, I decided to take part in the fellowship.

1. Learning - While not new to the blockchain space, I only had been focused on building NFT project and NFT marketplaces. Change is the only constant in life and I didn't want to be labelled as a one trick pony. The fellowship was the perfect opportunity to learn new concepts in DEFI and how a variety of projects in Ethereum worked.
2. Community - Not only did I have the opportunity to learn, I also got to do it together with a community of like-minded individuals.
3. Career advancement - Were I not in school, this would probably be top of this list. I knew the fellowship was going to be a catalyst in enabling me land a job or the next fellowship/internship.

</br>

## Pre-Fellowship Prep <a name="pre-fellowship"></a>

Pre-fellowship preparation can be a daunting task, but my experience having gone through the MLH fellowship taught me persistence and having clear goals that align with what the fellowship entails is key. An added advantage is seeking advice from a mentor or previous fellow to learn about what they look for when screening applicants. I would know 😂, having been rejected almost five times from the MLH fellowship. [あきらめかけているあなた (NEVER GIVE UP!)](https://youtube.com/watch?v=KxGRhd_iWuE)

As Eren once famously said,

> "Tatakae!"

During my preparation, I talked to Shriya from develfolio to determine my eligibility in the program(I'm Kenyan) and talked to a former Polygon fellow [Rakshit](https://twitter.com/rakshit087) who encouraged me to apply.

In the Wei application form, these are the field required to be submitted with some being optional

1. Your experience level building on Ethereum.
2. Your standout project.
3. Projects that you might be interested in building if accepted into the fellowship.
4. The reason you are applying for the fellowship.
5. Nationality. (checkbox on whether you are an Indian national)

From my experience I would say that getting through the submission stage, might be the easy part because now you will be invited for an interview and you must align what you talk about in the interview with what you wrote in the submission stage. Keep in mind that the submission stage ran for almost 2+ months.

I also know that my open source work contributed greatly in being accepted for the fellowship so build projects and contribute to open source. It does help.

</br>

## Fellowship Experience <a name="fellowship-exp"></a>

### Learning Phase <a name="learning-phase"></a>

My experience participating in EthIndia is very positive. In the first month, the learning phase, I brushed up on my Ethereum skills by working on [speedrun Ethereum](https://speedrunethereum.com/). I would highly recommend the course to new and veteran builders who might want to learn about Ethereum or brush up on their skills respectively.
In the first months, I learned so much about working on Ethereum smart contracts and documented my experiences on these two blogs, [week one and two](../eifweekoneandtwo) and [week three and four](../eifweethreeandfour)

We also had several developer hours with Denver from the Devfolio team to discuss progress learning about Ethereum. **Shoutout to [Garvit](https://twitter.com/plusminushalf)** for being an amazing mentor.

In addition to the developer hours and sessions we held with our mentors, the Devfolio team also organized learning sessions from projects from past EthIndia fellows and industry experts. These talks were centred around [DAOs](https://www.youtube.com/watch?v=88SxSgJK3Po&list=PLar2Ti_Qchk4BgWr5wcsROHD0dGFq7byj&index=3), [NFTs](https://www.youtube.com/watch?v=LhTuUR6q7JE&list=PLar2Ti_Qchk4BgWr5wcsROHD0dGFq7byj&index=7), [DEFI](https://www.youtube.com/watch?v=BX2ryqNsv3I&list=PLar2Ti_Qchk4BgWr5wcsROHD0dGFq7byj&index=2) and wallets in which my mentor Garvit, was working on a wallet implementation for [EIP-4337](https://eips.ethereum.org/EIPS/eip-4337).

Finally we got to showcase what we had learned from these sessions by doing research on a DEFI project and presenting it. My presentation on [1inch](https://www.youtube.com/live/DGW0mvJ15ic?feature=share&t=1529).

### Build Phase <a name="build-phase"></a>

In the second month of the fellowship, we entered the build phase. As the name suggests, during this phase we had come up with projects and build them. Remember in the form you filled you said that there are some things that you would have wanted to work on. This is where they come in.

In my case I was able to partner up with another fellow - [Pranay](https://github.com/heisenberg-737/) to work on, Kratoswap, a decentralized NFT marketplace with the use of an automated market marker. This would enable us to eliminate the need for user's to have to bid and have their bid accepted in traditional marketplaces. With an NFT AMM, their trades would go through instantly if enough liquidity was available in the requested pool. The idea was solid though the first challenge we ran into was that we could not use the traditional bonding curve such as `x * y = k` that was popularized by [UNISWAP V2](https://docs.uniswap.org/contracts/v2/concepts/protocol-overview/how-uniswap-works) since we cannot trade assets like NFTs that can only be 1 at a time where as this curve can never fully go to zero. We opted to use the [SudoSwap lssvm contracts](https://github.com/sudoswap/lssvm/) to help us counter some of these problems and deploying it to Polygon.

Pranay was also able to add a [quadratic bonding curve](https://github.com/Heisenberg-737/KratoSwap/commit/4c61dbfcae5a6de46edd3f18a6ec9cbadd0a0b96) that users to get in on a project early on when the price is still cheap as it gets more expensive as time goes on and the trading volume increases.

I can hopefully do a deep dive on NFT AMMs soon as this seems like a challenging but interesting topic to look into.

-   [Smart Contracts](https://github.com/Heisenberg-737/KratoSwap)

-   [Front End](https://github.com/jim4067/kratoswap-frontend)

-   [Back End](https://github.com/jim4067/kratoswap-backend)

<br/>

## Conclusion <a name="conclusion"></a>

This was a really exciting experience to be part of. Not to sound like a broken record, but I would highly recommend this fellowship to anyone wanting to get into Ethereum development or a seasoned developer working on an Ethereum related project.

> *you miss 100% of chances you don't take*