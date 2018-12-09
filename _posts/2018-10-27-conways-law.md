---
layout: post
title:  "Conway's Law"
date:   2018-10-27 16:30:00
categories: Methodology
tags: software product
---

* content
{:toc}

Melvin Conway introduced the idea in 1967
> Any organization that designs a system (defined broadly) will produce a design whose structure is a copy of the organization's communication structure.

or in a short form "Organization dictates design"

I first learned Conway's Law in 2014 at InfoQ ArchSummit, where Mike Amundsen gave a speech about Conway's Law at a distance. Now it's been 4 years, and I've seen a lot of evidence to prove the idea.

Following is an example. On the left side, three teams are involved to develop a product. Each team has only one kind of expert(in real world instead of there's only one kind of expert in a team, it's very likely that a team has limited access to certain resources. e.g. service team is not allowed to access security stuff) and will focus on a part of the product. The three components together is the final product. While on the right side, one team which has all kinds of experts, working on the same product.
From customers' point of view, in order to use the product developed by the left teams, customers need to talk to three teams to understand all the components. But to use the one developed by the right team, less communiation is needed.
![Organization and Product](https://github.com/precompiler/precompiler.github.io/raw/master/_posts/ConwaysLaw.jpeg)
