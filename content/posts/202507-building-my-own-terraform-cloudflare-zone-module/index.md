---
title: "Building My Own Terraform Cloudflare Zone Module (Because Most Were Still v4)"
description: "How and why I built my own Terraform module for Cloudflare zones after running into outdated v4-based modules, plus what changed in provider v5 and what I learned publishing my own module."
summary: "A practical write-up on creating a Cloudflare zone Terraform module when existing community modules were still tied to provider v4."
categories: ["DevOps", "Terraform"]
tags: ["Terraform", "Cloudflare", "Infrastructure as Code", "DevOps", "Open Source"]
date: 2025-07-22
draft: false
---

I hit a very common Terraform problem: I needed to move fast, but most of the Cloudflare modules I found were still built around provider v4.

At first, I thought I could just "adjust a few things" and keep going. But after enough friction, I decided to stop patching around other people’s assumptions and build my own module from scratch:

https://github.com/rashedobaid/terraform-cloudflare-zone

This post is a quick breakdown of why I did it, what changed with Cloudflare provider v5, and what I learned while turning it into a reusable module.

## Why I Built My Own Instead of Forking

Most existing modules I reviewed had one or more of these issues:

- tightly coupled to Cloudflare provider v4 resources and arguments
- opinionated in ways that didn’t fit my setup
- limited extensibility (especially around rulesets)
- hard to safely upgrade without breaking existing infrastructure

I considered forking, but a clean module gave me more control over:

- upgrade path to provider v5+
- module input design
- defaults and optional features
- long-term maintainability

In short: it was faster to own the design than inherit technical debt.

## What Provider v5 Changed (and Why It Matters)

The v5 provider introduced API and schema differences that made older modules feel fragile.

This wasn’t just a few breaking fields — it was a broad provider refactor. A lot of module assumptions from v4 no longer mapped cleanly to v5, especially once you started combining DNS records, Argo options, and rulesets.

I wanted a module that was explicitly designed for modern provider behavior, not one that accidentally worked through backward compatibility.

## Scope I Wanted from Day One

I kept the module focused on zone-level operations I use most:

- create a new zone or target an existing one
- manage DNS records consistently
- optionally enable Argo Smart Routing and Tiered Caching
- manage rulesets in a reusable structure

That gave me a practical base without trying to model every Cloudflare feature immediately.

## Module Design Decisions

A few design choices made this module easier to use and maintain:

### 1) Clear separation between required and optional inputs

Only the essentials are required (`account_id`, `zone`). Everything else is optional with sane defaults.

This keeps simple use-cases simple while still allowing advanced setups.

### 2) Feature flags for optional services

For Argo features, I used explicit booleans so it’s obvious when something is enabled.

That helps prevent accidental billing-impacting changes and makes plans easier to review.

### 3) Structured objects for records and rulesets

Instead of dozens of top-level variables, I leaned on typed object inputs for records and rulesets.

This keeps module calls readable and easier to extend over time.

### 4) Output values that are operationally useful

I exposed values I actually care about in real workflows, such as:

- zone ID
- nameservers
- ruleset IDs
- DNS record ID map

Outputs should help integration, not just mirror provider internals.

## Example Usage

A minimal call looks like this:

- set `account_id`
- set your `zone`
- pass your `records`
- optionally pass `rulesets`
- toggle Argo flags only when needed

You can see the full example in the README:

https://github.com/rashedobaid/terraform-cloudflare-zone#example

## Lessons Learned While Building It

### Keep the interface stable, not just the implementation

When people consume a module, your variables and outputs become your API. Changing those casually breaks downstream users fast.

### Prefer explicit toggles over hidden magic

Especially for Cloudflare features with cost/performance implications.

### Write for real usage, not theoretical completeness

A smaller, well-documented module that handles common cases is better than a giant one nobody trusts.

### Provider upgrades are design events

Moving to v5 isn’t just a version bump. It’s a chance to simplify assumptions, tighten typing, and improve module ergonomics.

## If You’re Stuck on v4-Based Modules

If your existing module ecosystem is still v4-oriented, you basically have three choices:

1. stay pinned and accept technical debt
2. fork and gradually modernize
3. build a focused module around your real requirements

I went with option 3, and for my use-cases it was the right call.

## Final Thoughts

This module started as a practical fix for upgrade friction, but it ended up being a solid foundation I can reuse across projects.

If you’re managing Cloudflare with Terraform and feel boxed in by old module designs, building your own module is absolutely worth considering.

Sometimes the fastest path is owning the abstraction.

---

If you want to use or review it, here’s the repo again:

https://github.com/rashedobaid/terraform-cloudflare-zone
