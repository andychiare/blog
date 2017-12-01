---
layout: post_extend
title: "Why Auth0 chose Serverless Extensibility"
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
  image: https://cdn.auth0.com/website/blog/extend/why-auth0-chose-serverless-extensibility/logo.png
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

However, how did Auth0 get to the point of offering Auth0 Extend as a product? It has been a four-year journey that predates both [Amazon Lambda](https://aws.amazon.com/lambda/) and the term Serverless. Like all innovation, it starts with resource constraints, the need to give customers the features they wanted and making sales.

## The Problem We Were Trying to Solve

In the early days at Auth0, there were two groups: Core and Field Engineering. Core focused on the core functionality of the authentication product, and the field engineers helped customers use the product in their applications. The company was discovering what customers needed in the product.

Our customer's focus was on the authentication transaction. A lot of interesting features can attach to the process of someone is trying to login:

- Profile Enrichment
- Progressive Profiling
- Authorization
- Claims Transformation

The possibilities are endless. We could not possibly build every feature at once.

Moreover, sometimes building features into the core product for customers is not the right idea. We needed a way to try ideas out. An idea may sound great, but in practice be problematic. Investing core engineering resources to these ideas is very expensive.

> **"The primary use case for extensibility at Auth0 was to empower field engineers to work on the last mile solutions for the customer without involving core engineering."**<br />
> Eugenio Pace - Co-Founder, VP Customer Success

If every interaction with a customer involved identifying an idea, putting it in the backlog and coming back to them at a later date, it would add friction to the process and turn customers away.

## Custom Code Extensibility

The inspiration for custom code extensibility as a solution came from spreadsheets. Excel gives you much functionality out of the box, but there is always a function, macro or calculation that is not there. However, you can write them yourself directly in Excel removing the dependency on Microsoft engineers.

We wanted a similar experience for our users. A user should be able to log in to the dashboard, write a small amount of node.js code that executes later during authorization transactions.

Unlike webhooks, this experience does not expose your customer to the necessity of setting up servers to run a service. They can come in write their business logic in the textbox, debug it in place and put it in production. The motivation was keeping it extremely simple.

The earliest MVP version of this concept used [node sandbox](https://www.npmjs.com/package/node-sandbox
). It was very similar to a CGI model; spinning up a separate node process, send the rule code to execute in it and collecting the result. Execution all happened one process per authorization transaction.

This implementation had some issues, primarily a lack of security. It was merely creating a process boundary between the core Auth0 stack and the customer's code. Node sandbox was primarily preventing well behaved, well-intentioned code from accidentally bringing down the authorization service or other sandboxed code.

However, the MVP proved that the customer experience aligned well with our philosophy of "Identity made simple for developers."

## How do you secure execution of untrusted code?

> **"When Eugenio and I started talking in August of 2014, this turned out to be my interview question. We have this problem, this is what we want to do, how would you secure it? "**<br />
> Tomasz Janczuk - Chief Webtask Architect

The initial high-level goal for the Webtask architecture was to create a protocol boundary around the core Auth0 stack and the custom code execution engine and create a multitenant sandbox behind that protocol.

The first prototype of the Webtask architecture was an interview exercise. Extensibility was a pressing enough problem; it turned into an actual assignment. The implementation was relatively simple; it was a month of work to complete.

The stabilization effort was something entirely different. You cannot solve a stabilization problem by adding people to the team.

## Iterating on the Webtask Stack

The first version of Webtask used early versions of [Core OS](https://coreos.com). Core OS at the time used three distinct technologies to provide a platform for creating distributed systems:

- Docker for OS containerization
- etcd for distributed configuration management
- Fleet for distributed service management

Core OS promised that these three components complemented each other well. Docker organized compute, etcd provided configuration management in a cluster and Fleet decided what workload ran where and when.

This architecture maintained a named container on a single virtual machine within a cluster. When a request came in to execute some code in this container, we used etcd to find the VM the container was provisioned on and reverse proxy the request to that VM.

Becuase this architecture depended on distributed configuration management it had a higher exposure to failure. Also,  add to this the fact that individual virtual machines had a certain probability of failure for random reasons. If a single container only lives on a single virtual machine, there is a single point of failure for that container.

Once in production, we realized we were relying on the cutting edge of three technologies. We had instances of the platform destabilizing at 2:00 AM in the morning in Austraila one too many times. It felt like a constant game of whack-a-mole, chasing one stability issue after another. When we upgraded the stack to new versions of Docker, etcd or Fleet; some new issue would pop up.

The focus of version two of Webtask was to simplify the architecture and remove every anything that was not absolutely needed to increase the fault tolerance of the overall system.

We started with the assumption that the virtual machines should operate entirely independently making them private universes. In terms of container management, the VMs have no business communicating with each other for anything but non-critical functionality like real-time logging.

This change removed the need for etcd because we no longer needed to exchange state between the virtual machines. It also removed the need for Fleet because a named container was allowed to exist on all the virtual machines in the cluster at any given time.

When a request comes in, the load balancer decides to send it to a particular virtual machine. The virtual machine considers that request in complete isolation from the other VMs in the cluster. If the VM needs a container, it will provision it on demand.

At this point the only component of Core OS still in use was Docker. So, we dropped down to vanilla Ubuntu. The process of simplification was a metamorphosis of the sack that resulted in considerable improvements in stability.

The third version of the Webtask stack focused on real-time logs. To this day it is the only feature in the Webtasks architecture that requires virtual machines to be aware of each other's existence.

Real-time logs work by consolidating logging information into a single point. A client makes a management API request providing filtering information. Regardless of the virtual machine in the cluster that the client attaches to, it collects the real-time logging information from all other VMs and streams it back to the client on the HTTP response.

The original implementation of this used [Kafka](https://kafka.apache.org/). Kafka is a very high throughput message broker optimized for log aggregation scenarios. There are many success stories from places like Netflix using it.

Kafka builds on top of [ZooKeeper](https://zookeeper.apache.org/) for distributed configuration management, similar to how Core OS uses etcd. It turned out, at the time, ZooKeeper had a few skeletons in the closet regarding stability. The Kafka ZooKeeper components were destabilizing enough to cause virtual machine failures. As with etcd, tracking down the issues was never-ending after three months.

We started looking for alternatives and landed on [ZeroMQ](http://zeromq.org/). ZeroMQ has no storage involved at all. It is an in-memory system that provides messaging patterns over TCP. ZeroMQ's publish/subscribe pattern allowed the source of logging information to act as a publisher. We could have any number of subscribers providing filtering criteria and receiving messages. One nice feature is if there are no subscribers ZeroMQ merely drops messages on the floor;  which was what we needed for real-time logging.

Switching to ZeroMQ was the single most stabilizing change we made in the history of the Webtask cluster. It was such an impressive improvement Tomasz wrote a [post about it](https://tomasz.janczuk.org/2015/09/from-kafka-to-zeromq-for-log-aggregation.html). That post received 10,000 views the first day it published. Others were apparently having similar issues.

## Summary

Adding extensibility to the product allowed field engineers to say "yes" very often and show those customers a way to accomplish their goals. It opened up a  window of customization where field engineers could work independently from core engineering. They could deliver customizations very quickly without waiting weeks or even days.

QUOTE: In some cases, we were able to deliver demos where we implemented feature requests in the demo during the meeting. It was amazing for field engineers and our customers. - Eugenio Pace

Putting extensibility into the hands of customers also gives us great insights into where the market is going. If we see an extension being implemented again and again through the use of custom code, that is a validation of that feature and an opportunity to add in the core product.

Multi-Factor Authentication came about this way. MFA was not a switch on the dashboard initially. It was merely a rule you applied to your account. Over time, our customers' usage of this rule indicated MFA was an important feature to have in the core product.