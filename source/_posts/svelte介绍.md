---
title: svelte介绍
date: 2019-11-14 14:11:02
tags: [JavaScript]
---
[virtual-dom-is-pure-overhead](https://svelte.dev/blog/virtual-dom-is-pure-overhead)

Unlike traditional UI frameworks, Svelte is a compiler that knows at build time how things could change in your app, rather than waiting to do the work at run time.

The danger of defaulting to doing unnecessary work, even if that work is trivial, is that your app will eventually succumb to 'death by a thousand cuts' with no clear bottleneck to aim at once it's time to optimise.

Svelte is explicitly designed to prevent you from ending up in that situation.

#### Why do frameworks use the virtual DOM then?
It's important to understand that virtual DOM isn't a feature. It's a means to an end, the end being declarative, state-driven UI development. Virtual DOM is valuable because it allows you to build apps without thinking about state transitions, with performance that is generally good enough. That means less buggy code, and more time spent on creative tasks instead of tedious ones.

But it turns out that we can achieve a similar programming model without using virtual DOM — and that's where Svelte comes in.
但是事实证明，我们无需使用虚拟DOM就可以实现类似的编程模型-这就是Svelte的用武之地。