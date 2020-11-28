---
layout: post
category: development
title: Nodejs benchmark WSL vs Windows
tags: [WSL, nodejs, benchmark]
---

## What is better environments for nodejs?

Recently I needed to setup nodejs, because recently I bought the new desktop. In my old system, I suffered from slow `npm install`. Because I don't want to use preview version of windows, I can't use WSL2. So I have two options for nodejs environment. Nodejs on WSL or Native Windows. For right decision, I measured the time for some of items related with nodejs.

## Test Environment

* CPU: Ryzen 3700x (8C 16T)
* RAM: 64GB 2600Mhz (Samsung 16GB x 4)
* SSD: Samsung 970 EVO
* WSL: Arch Linux WSL version 1
* nodejs: v12.11.1

## Test Result

| Category           | Native Windows   | WSL           | Comparison
|--------------------|------------------|---------------|---------------|
| Create React App   | 94s              | 147s          | WSL takes 56% more.
| npm run build      | 6s               | 6s            | same
| Octane (3x avg)    | 41641            | 41438         | same (0.4% diff)

## Conclusion

* When it comes to computation, WSL and Native Windows are almost same.
* Because WSL's IO is not good, CRA(Create React App) takes 56% more in WSL.
* But in WSL2, IO performance will be better. I am looking forward to 20H1 update.
* Because of Linux's convinient CLI environments, **I will choose WSL** despite of the preformance drop.
