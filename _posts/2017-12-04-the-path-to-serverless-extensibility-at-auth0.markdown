---
layout: post_extend
title: "The Path to Serverless Extensibility at Auth0"
description: ""
date: 2017-12-04 10:19
category: Extend, Business
author:
  name: "Bobby Johnson"
  url: "https://twitter.com/NotMyself"
  mail: "bobby.johnson@auth0.com"
  avatar: "https://cdn.auth0.com/website/blog/profiles/bobbyjohnson.png"
design:
  bg_color: "#3445DC"
  image: https://cdn.auth0.com/website/blog/extend/securing-webtasks-part-1-shared-secret-authorization/webtasks.png
tags:
  - extend
  - serverless
  - webhooks
  - extensibility
related:
  - 2017-05-16-introducing-auth0-extend-the-new-way-to-extend-your-saas
  - 2017-05-19-serverless-webhooks-with-auth0-extend
  - 2017-08-22-for-the-best-security-think-beyond-webhooks
---
Previously, we wrote about the emerging pattern of [Serverless Extensibility](https://auth0.com/blog/why-is-serverless-extensibility-better-than-webhooks/). Examples of the concept are popping up in many of the services you use daily like [Twilio](https://www.twilio.com/functions), [Stamplay](https://stamplay.com/) and here at [Auth0](https://auth0.com/).

Serverless Extensibility is a logical extension to Webhooks. We have created a product call [Auth0 Extend](https://auth0.com/extend/) based on the pattern that allows other SaaS companies to offer it from their products quickly.

When [Jeff Lindsay](https://twitter.com/progrium) was introduced to Extend he certainly felt we were on to something. Jeff was the person who coined the phrase [Webhook](http://progrium.com/blog/2007/05/03/web-hooks-to-revolutionize-the-web/), which has become ubiquitous on the web as a way to offer extension points from applications online.

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">This was the whole point of pushing webhooks in 2007. Literally built this as a prototype. <a href="https://t.co/Wyoz0qXO9P">https://t.co/Wyoz0qXO9P</a></p>&mdash; Jeff Lindsay (@progrium) <a href="https://twitter.com/progrium/status/864588610858881029?ref_src=twsrc%5Etfw">May 16, 2017</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

But how did Auth0 get to the point of offering Auth0 Extend as a product? How did we get here? It has been a four-year journey that predates both [Amazon Lambda](https://aws.amazon.com/lambda/) and the term Serverless. Like all innovation, it starts with resource constraints, the need to give customers the features they wanted and making sales.

## The Problem We Were Trying to Solve

In the early days at Auth0, there were two groups: Core and Field Engineering. Core focused on the core functionality of the authentication product, and the field engineers helped customers use the product in their applications. The company was discovering what customers needed in the product.

Our customer's focus was on the authentication transaction. A lot of interesting features can attach to the process of someone is trying to login:

- Profile Enrichment
- Progressive Profiling
- Authorization
- Claims Transformation

The possibilities are endless. We could not possibly build every feature at once.

Moreover, sometimes building features into the core product for customers is not the right idea. We needed a way to try ideas out. An idea may sound great, but in practice it is problematic. Investing core engineering resources to these ideas is very expensive.

The inspiration for extensibility as a solution came from spreadsheets. Excel gives you much functionality out of the box, but there is always a function, macro or calculation that is not there. However, you can write them yourself directly in Excel removing the dependency on Microsoft engineers.

The primary use case for extensibility at Auth0 was to empower field engineers to work on the last mile solutions for the customer without involving core engineering.  If every interaction with a customer involved identifying an idea, putting it in the backlog and coming back to them at a later date, it would add friction to the process and turn customers away.

Adding extensibility to the product allowed field engineers to say "yes" very often and show those customers a way to accomplish their goals. It opened up a window of customization where field engineers could work independently from core engineering. They could deliver customizations very quickly without waiting weeks or even days.

> **"In some cases, we were able to deliver demos where we implemented feature requests in the demo during the meeting. It was amazing for field engineers and our customers."**<br />
> Eugenio Pace - Co-Founder, VP Customer Success

Putting extensibility into the hands of customers also gives us great insights into where the market is going. If we see an extension being implemented again and again through the use of custom code, that is a validation of that feature and an opportunity to add in the core product.

Multi-Factor Authentication came about this way. MFA was not a switch on the dashboard initially. It was merely a rule you applied to your account. Over time, our customers' usage of this rule indicated MFA was an important feature to have in the core product.