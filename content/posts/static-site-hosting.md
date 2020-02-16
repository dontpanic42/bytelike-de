+++
title = "Hosting a static site on AWS S3"
date = 2020-02-16T08:32:58.029Z
author = "Daniel"
cover = ""
tags = ["Tech", "AWS", "Hosting on S3", "S3", "CloudFront", "ACM", "Route 53"]
keywords = ["Tech", "AWS", "Hosting", "S3", "CloudFront", "ACM", "Route 53"]
description = "Let's have a look at how we can host a simple static website (like this blog) on AWS S3."
showFullContent = false
+++

When you spend your working days designing and building enterprise IT, this tends to have an
effect on your private projects. So when I (once more) decided I needed a website, my first
thought - naturally - was: Ugh, now I have to setup the Kubernetes cluster, set up auto patching
(or even better - an AMI building factory) for the worker nodes (or go to EKS directoy?), where will 
host the CI/CD server? ...

I quickly realized that it would take me days or weeks of tinkering after work to set it all up. And then I took a step
back and realized that everything I needed I could just do with a simple static site and a site generator
like [Hugo](https://gohugo.io/). And from a hosting perspective, when you hear "static site", as an AWS guy,
you tend to immediately think "S3".

Turns out hosting with S3 is a little more involved than just enabeling the "Static website hosting" property and upload
your stuff (since SSL is still not supported with a custom domain...). But still doable on a rainy afternoon. So this
is the design I came up with:

![Static hosting architecture](/img/posts/static-site-hosting/architecture.png)

The "trick" when hosting on S3 is to use [Amazon CloudFront](https://aws.amazon.com/cloudfront/). For the uninitiated, CloudFront is a Content Delivery
Network (CDN)-as-a-Service by AWS. It does caching and moves your endpoint nearer to your user by serving your
content at one of their many Points of Presence all around the world. While this is nice to have, it does not
matter much in my usecase. What *does* matter is that it allowes you to use a custom Domain and SSL Certificate and
also happyly servers files originating in an S3 bucket.

My domain name was registered through AWS, so I can just use [Route 53](https://aws.amazon.com/route53/) to create the A (IPv4) and AAAA (IPv6) records pointing to CloudFront. You get (public) SSL Certificates for your domain for
free when you use [AWS Certificate Manager](https://aws.amazon.com/certificate-manager/), so this was a no-brainer.

One caveat when using CloudFront with Certificate Manager is that when you use another Region then `us-east-1`,
you won't be able to select your certificate when setting up CloudFront. So make sure your certificates
are always created in N. Virigina (`us-east-1`), even when the rest of your setup lives else where (`eu-central-1`
in my case).

Keep your eyes open for the next posts, in which we will discuss the technical details of implementing our
architecture as-code with Terraform, automating our setup with Github Actions and create a simple site with Hugo.
