---
layout: post
title: "Github Webhooks"
slug: "githubWebhooks"
date: "2023-01-06"
comments: false
categories:
  - devops
tags:
  - github
  - continuous deployment
  - webhooks
---

Github webhooks is a feature that allows you to send HTTP POST payload to certain events that occurs in your repository hosted on [github.](https://github.com) Events like pushing to a branch or merge are good examples for using webhooks. 

[Read more about webhooks](https://https://docs.github.com/en/developers/webhooks-and-events/webhooks/about-webhooks)

Webhooks can be used for triggering builds or deployments and allows us to integrate devOps principles even if you don't use any Platform as a Service(PaaS).

Here is a really good blog post of setting up webhook for continuous deployment in your VPS.

[Setting up Github Webhook](https://maximorlov.com/automated-deployments-from-github-with-webhook/)

> I use webhooks to trigger a hugo build on my server and move the contents of the static site generated to my nginx root folder. 