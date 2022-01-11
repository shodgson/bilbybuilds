---
title: "Speed build template"
date: 2021-11-08T10:41:54+11:00
draft: false
---

## Context
I need a quick way to setup businesses, particularly software-as-a-service (SaaS).

My last [speed build]({{< ref "/speedbuilds/linky-dink" >}}) took 57 hours to build. A lot of that are stock-standard components that should be reusable (e.g. 11 hours spent on authentication, 4 hours spent on payment integration).

I have tried a few frameworks (Firebase, Amplify), but they eventually let me down when I want to implement a feature they do not support.
I am not a fan of many frameworks. They add a level of abstraction that:
1. Require learning something new
1. Impose restrictions, eventually
1. Add complexity in maintenance

Instead, I like _templates_. So the aim here is to create a simple SaaS repository that I can clone anytime I start a new speed build.

## Requirements:
A basic app that allows users to create an account and increment their counter.
The counter will only be allowed to exceed 5 if the user pays a monthly subscription fee.

The app will have:
- Multiple environments
  - Local development environment
  - Production environment
- Web UI
  - Simple landing page
  - Basic app
- Basic database
- Authentication and authorisation
- Payment
- Version control
- Testing

Out-of-scope:
- Real-time updates (e.g. websockets)

## Summary
Stack:
- Backend
  - Compute
    - AWS: Lambda (written in Go v1.17)
    - AWS: API Gateway (v2/HTTP)
  - Database
    - AWS: DynamoDB
  - User identity
    - AWS: Cognito
  - Payments
    - Stripe (checkout)
  - Tooling
    - Serverless v2.57
- Frontend
  - Landing page
    - Vanilla HTML/CSS/JS
  - App
    - Vue v3.2, Vue-router v4.0, Vuex v4.0
    - Tailwind v2.2, DaisyUI v1.16
  - Tooling
    - Vite v2.1
    - TypeScript v4.4
- Tooling
  - GitHub

## Guide
1. Buy a domain, e.g. from Namecheap
1. Create a free email account on Gmail
1. Create a new AWS root account
1. Create a new AWS IAM user
  1. Then create a profile in .aws/credentials
  1. Update .env to include profile name
1. Setup a Hosted Zone in AWS' Route53 
1. Set domain's nameservers to AWS on Namecheap
1. (Optional) Set up a mail server with Zoho
1. Create an S3 bucket and configure as a website (see script)
1. Create a Stripe account and a subscription product
1. Jump through hoops to deploy:
  1. Deploy to production
```
make deploy STAGE=${STAGE}
```
  1. Create a Stripe webhook with `customer.subscription.created` and `customer.subscription.deleted` events sent to the appropriate API address (e.g. https://{api-id}.execute-api.{region}.amazonaws.com/subscription/webhook)
  1. Update .env.production with the output from the previous command and the `Signing secret` from the Stripe webhook (e.g. whsec\_abc123)
  1. Reploy backend to production, build frontend, and then deploy frontend to bucket created above
```
make deploy STAGE=production
make buildfront
make deployfront AWS_PROFILE={profile} BUCKETNAME={bucketname}
```
The site should be available at http://{bucketname}.s3-website{. or -}region}.amazonaws.com
1. Create a cloudfront distribution to direct requests to the domain to the S3 bucket

## Optional extras that are low effort
- Move API to the same domain or subdomain
- Send emails from own domain for Cognito
