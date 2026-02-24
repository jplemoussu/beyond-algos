---
layout: post
title: "Start here"
date: 2026-02-09
---

## Why write a blog?

As I write this in 2026, I've been working on an automated platform to manage my equities investment portfolio for over four years. The idea behind this blog is to record my journey, organise my thoughts, and perhaps more importantly, provide a guidebook to my children so they don't repeat the same mistakes I made.

With the rise of agentic coding, by leaving a trace of my learnings and thought process, anyone with access to the codebase should be able to maintain it or rewrite it in whatever language is popular in the future. Technology changes, but financial markets are a necessity to any successful economy. The underlying human behaviours that drive them - greed and fear - are timeless. The investment and risk management principles I've implemented should remain applicable for a long time to come.

## What content will I write about?

Not sure! I can tell you what I *won't* write about: soulless LinkedIn self-promotion slop. There are plenty of fake engineers out there who rehash the same spaghetti code tutorials, pretending to be an expert in whatever tech is buzzing this week. Instead, I'll focus on core concepts for understanding financial markets and designing systematic trading systems, with some reflections along the way - hence the name "Beyond Algos". I'm not trying to sell you anything. I'll be candid about my mistakes and uncertainties, updating and referring back to old posts as I learn.

## What got me into equity markets?

Whether you like it or not, financial markets control your quality of life. I learned this early in my career thanks to an angry old man: Carl Icahn. In a weird butterfly effect, his actions to push eBay to spin out PayPal and boost his newly acquired stake in the company caused a chain reaction that resulted in me losing my job. During the reshuffle, they decided to close APAC product management and centralise it back to the US. My role was eliminated. 

In typical Silicon Valley HR style, I came up to the office one morning, got summoned to a meeting room on a different floor, and was asked to hand over my access card. Perfect timing - my wife was pregnant with our first, and we'd just started repayments on our first home.

On the positive side, after seven years I'd accumulated a bunch of PayPal shares which were suddenly gaining in value due to rising interest in the fintech industry. On second thought, maybe I should also become an angry shareholder, rather than be at the mercy of one. It was clear I had to figure out this stock market thing, using my employee shares as starting capital. Having experience in the software industry, I decided to focus on the US market since the ASX is mostly banks and miners, the latter I knew nothing about.

## Why did I build my own trading system from scratch?

In a word: naivety. If only I knew how hard building a near real time trading system would be, I would have bought an index fund and called it a day. At the time there were no off-the-shelf solutions that could do what I wanted: trade with a risk-management-first mindset.<sup>1</sup> 

I started trading manually, mostly using technical analysis<sup>2</sup> and basic fundamental research. Due to timezone differences, trading the US market from Australia is exhausting. Reviewing thousands of charts, setting up buy/sell orders, and hoping whatever I bought would be up when I woke up.

What I didn't realise is that investing is essentially a mentally draining chain of decisions: Buy? Sell? Do nothing?

Why couldn't I offload all this mental load to a computer? Surely I could learn Python, run some pandas operations, and watch the balance increase? Keep reading to find out how that turned out.

<sup>1</sup>*I believe that's still the case today. QuantConnect (founded in NZ) offers a comprehensive solution, but focuses more on backtesting and research rather than real-time risk management and execution. It's fairly pricey and the entry plan limits you to two live strategies. I'm also wary of fully relying on a third party in a different jurisdiction to manage my money - there's no guarantee any platform you use today will be around in 10 years.*

![Technical analysis](/beyond-algos/assets/images/start-here/ta-dinosaur.webp)
<sup>2</sup> *Technical analysis in a nutshell. A lot of it is made up nonsense, but some concepts like momentum and trendlines hold up and are essential for algorithmic trading.*

