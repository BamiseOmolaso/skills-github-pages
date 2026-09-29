---
title: "Building My Cloud Portfolio the Hard Way"
date: 2026-09-29
---

I'm a medical doctor who got curious about how production systems actually work, so I built a portfolio website and then rebuilt it properly.

## Version 1: the easy way

The first version took one weekend. I built it with Next.js, hosted it on Vercel, used Supabase for the database, and it was live. It worked, but I didn't learn much from it.

## Version 2: thinking like a cloud engineer

I took it apart and started again, this time asking how a solutions architect would build it. The stack:

- **Frontend and app layer:** Next.js 14 (TypeScript, Tailwind), containerized with Docker
- **Compute:** AWS ECS Fargate behind an Application Load Balancer
- **Data:** PostgreSQL on RDS (through Prisma), plus Redis for rate limiting and caching
- **Secrets:** AWS Secrets Manager, so nothing sensitive lives in the repo
- **Infrastructure as Code:** Terraform, with separate dev, staging and prod environments
- **CI/CD:** GitHub Actions with OIDC into AWS (no long-lived access keys). Tests, type checks and security scans run before every deploy.

Is that overkill for a portfolio? Yes. But every layer maps to something I'd use on a real job.

## What broke (and what it taught me)

Most of what I learned came from bugs:

- A container health check that fought with the load balancer's health check
- Outbound emails that linked to `localhost` in production
- A CI pipeline that kept trying to destroy my HTTPS setup
- Redis connections timing out after the stack had been paused for a long time

Each one showed me a way production differs from "works on my machine."

## The $1.50/month trick

Left running all the time, this architecture costs roughly $200–250 a month. So I wrote a `pause.sh` script. It destroys the load balancer, scales ECS down to zero and stops the database, but keeps the data, images, secrets and Terraform state. Running `resume.sh` brings everything back in a few minutes.

What it taught me: **"production-grade" doesn't have to mean "production budget."** The architecture stays the same. What changes the cost is the redundancy dials you choose to turn.

## What's next

The code is on GitHub: [BamiseOmolaso/cloudportfoliowebsite](https://github.com/BamiseOmolaso/cloudportfoliowebsite). In future posts I'll go through each decision in more detail: why I moved from Supabase to RDS, why I set up OIDC, and how the Terraform is structured.
