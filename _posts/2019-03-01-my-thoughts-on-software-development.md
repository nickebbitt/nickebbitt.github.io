---
layout: post
title:  "My thoughts on software development"
date:   2019-03-01
categories: agile, devops, java, tdd
comments: true
---

Around the beginning of this year as I was transitioning between jobs I decided to start writing down my views on software development. 

The majority of the ideas, approaches, principles and values in here are things I've learned about, experienced or simply formed an opinion on over the last 5 years or so.

This post is basically a dump of those thoughts.

There's likely nothing new here - I've learnt all I know from the amazing people I've worked with, consuming many books & videos (both software related and not), attending conferences and simply engaging with the many talented people in the various tech communities I'm involved with such as the Manchester Java Community. 

Apologies if I've not referenced quotes or ideas where I should have - if so please let me know and I'll update the post.

This is mainly a reference for future me, I may come back and update it from time to time as my thoughts evolve. It'll be great if anyone else finds it useful too.

* Will be replaced with the ToC
{:toc}

# Culture

## Prioritise the well being of the team

The well being of the team is more important than delivery. A happy and healthy team will likely result in good delivery anyway.

No one should feel undue pressure to deliver change at the expense of their physical or mental health.

No one should be expected to sacrifice their personal time to meet deadlines. This indicates a problem with the process, not the person.

## Ownership

The team owns the process.

Everyone on the team should feel empowered to challenge the status quo and lead the team with new ideas.

The absence of senior team members should not prevent decisions on what/how to get things done. Standard events such as stand-ups/retros/showcases should still happen due to a collective ownership of these things.

Encourage ideas and leadership qualities from all team members.

Distribute responsibility for organising/running things such as:
- showcases
- retros
- [code review](#code-review) sessions / katas

## Working software

At all times, the goal is for the application to be releasable.

We should always be able to show what we have built.

Demonstrate value, regularly.

## A safe environment

The team environment should be a safe space.

Failure is not a problem, it's a learning opportunity.

If a failure has an adverse impact, then it's a failure of the process, not the individual.

We should encourage experimentation, take calulated risks, and learn from the feedback.

## Collaboration

Wherever possible, find ways to work together on tasks. The value realised through shared knowledge, early/regular feedback and the general good team spirit this helps to create should outweigh any perceived short-term benefits of parellelising tasks.

## Support Driven Engineering
See [Support Driven Engineering (SDE) – Will Gallego](http://willgallego.com/2018/12/09/support-driven-engineering-sde/).

Team members who want to help those around them succeed are invaluable. Encourage everyone to have this mindset.

A key sign of a "senior" engineer is one who works hard to ensure those around them can achieve what they need to.

This can be through picking up the gnarly tasks that others avoid, or just unblocking their team mates when they get stuck, or in many other selfless ways.

## Personal development

Everyone is responsible for their own learning and development. Good companies and managers will help you fulfil your ambitions and reach your goals. They can't if you don't tell them though.

# Process

Central to any software development process is risk management. The [risk-first](https://github.com/risk-first/website/wiki) wiki provides an interesting analysis of various apsects of software development and how they compare in relation to risk.

## Simple, Valuable, Incrementally

> Keep It Simple, Make it Valuable, Built It Piece by Piece
> **Ron Jeffries**

> We try to develop the simplest thing that could possibly be useful, deliver it, and get feedback. Then we add to that.
> **David Tanzen**

## Planning

Some upfront design is essential.

Collaborative design sessions.

Produce useful artifacts. See [The C4 model for software architecture](https://c4model.com/).

Feature slicing, make sure deliverables are a good size. See [Small changes](#small-changes).

## User Stories

Whether user stories are captured using post-its on a wall, Trello cards or JIRA tickets, they should be created as a placeholder for a discussion. They shouldn't contain detailed requirements.

This helps to ensure the necessary conversations and detailed anaylsis occur as near as possible to when the story is actually worked on. This is particularly important as often storties in a backlog never get picked up so why waste time performing detailed analysis on them.

It also helps to ensure that the people who work on the story are the same people who understand the detail and have contributed to the decisions made.

## Feedback loops

Short feedback loops are key to being productive.

Examples:
- IDE's set up to provide feedback on style etc.
- Very fast unit tests.
- Customer involved in process to provide regular feedback.

Learning opportunities, the sooner we can learn something new about our product/process/etc., the sooner we can make a change and iterate.

## Daily stand-ups

Key for team synchronisation, should be quick, 1 min max per person. Break-out meetings where necessary.

Focus on the value.

## Showcases

Regularly demonstrate the changes the team has built, or lessons learnt to other teams, stakeholders.

## Retrospectives

Regular retros are very important.

**TODO:** research/practice running effective retros and ensure outcomes are actionable and met.

## Estimates

Estimating based on time is a waste, instead make stories/changes (feature slices) small enough to deliver a slice of value in 1-2 days.

Esimate the size of a feature. Use experience. Use t-shirt sizes e.g. Small, Medium or Large.

Only start development work on features once they have sliced into Small tasks. See [Small changes](#small-changes).

Choose the most valuable features first. Always ensure priorities are driven by business value.

## Small changes

The smaller the changes we commit/merge to master the better.

Reduce every change proposal to its absolutely minimally viable form. See [Minimally Viable Change](https://about.gitlab.com/handbook/product/#the-minimally-viable-change).

Why?
- smaller changes easier to review
- less risk
- easier to test
- easier understand
- easier to reason about

## WIP limits

Always allow time to do the right thing i.e write tests, refactor, design.

Use Work in Progress (WIP) limits to ensure the team doesn't take on too much work, and always has space for unplanned work that may (and will) arise.

WIP limits can be good to promote pairing e.g use a WIP limit that is half the number of team members.

## Handling blockers

Blocked tasks are those where it is unable to progress due to lack of skills or knowledge that are available within the team.

Waiting tasks are those that are dependent on an external party/process to the team e.g. regulatory sign-off etc.

> Blocking != Waiting

Encourage individuals to feel responsible for all tasks, not just the ones they are assigned to.

If a task is blocked, expect team to work together to unblock through knowledge sharing, pairing or mobbing.

Team should swarm on blocked tasks to ensure they are unblocked ASAP.

## Progress

The best indicator of progress is new features running in production.

Working closely with the customer reduces the need to produce detailed reports relating to progress.

Interesting metrics to observe are:
- the time from when a task is picked up to running in production
- how long tasks sit in a "blocked" (i.e. under our control)  state
- how long tasks sit in a "waiting" (i.e. out of our control) state

See [Handling blockers](#handling-blockers)

## Architectural Decision Records (ADRs)

> We will keep a collection of records for "architecturally significant" decisions: those that affect the structure, non-functional characteristics, dependencies, interfaces, or construction techniques.

See [Documenting Architecture Decisions](http://thinkrelevance.com/blog/2011/11/15/documenting-architecture-decisions) introduced by [Michael Nygard](https://twitter.com/mtnygard).

# Technical Practices

## Trunk-based development
See [Trunk Based Development](https://trunkbaseddevelopment.com/).

Integrate often with master.

No shared branches, or short-lived private feature branches ok (no more than 1-2 days).

Release branches ok.

Why?
- smaller batches, easier to merge
- merge often to reduce conflict
- if PRs, easier to review

Continuous code review required to support this.

## Pairing

Encourage pairing, limit [WIP](#wip-limits) to assist this.

Don't enforce 100% pairing, allow space for individual work.

Pairing works well but can be mentally draining.

Keep it fresh by rotating roles and pairs.

## Test-Driven Development (TDD)

A very useful but difficult to master technique.

Having tests is the most important thing. Writing them first should help us to produce better designs.

TDD is less about testing, more about using examples to drive out the design of an application

Needs investment as a team to use a test-driven approach to feature development.

Identify where engineers need help via pairing or training.

## Test-doubles / mocking

While frameworks such as Mockito are really powerful and can be useful to assist with testing, prefer a simpler approach to test-doubles e.g. for simple use-cases, roll your own stubs or spys.

Don't over use "mocks", usually testing more real things is more valuable.

Doubles can be really useful as the design of an API emerges using a [TDD](#test-driven-development-tdd) approach.

## End-to-end tests

Exercising end-to-end use-cases to verify the application is really useful. These tests should be written to reflect real usage of the system from a black box perspective.

## Code review

When pairing/mobbing, code review is a continuous interactive process between those involved.

Sometimes it can be useful to review code as a team. The collaborative process of reviewing a section of code in the system that has proven difficult to work with can work well. Make sure it is a safe space and those who wrote the code are comfortable with the process.

No blame should be associated with poorly written code. It is simply a learning opportunity.

## Development spikes

If we are uncertain about how to deliver a feature, we should use time-boxed development spikes to explore the options.

Aim is to learn quickly whether an idea has legs, if so invest more time to do it properly, otherwise pivot and try something else.

## Continuously Deploy Changes

Every change should be deployable. We can choose when to release.

Fully automated process should be capable of building & testing every commit, provisioning test environments, executing automated tests of various varieties to gain confidence.

Every change to the software should be deployed.

Ship continuously, not just at the end of a sprint.

This proves we can deploy when needed and helps to ensure when the pressure is on (e.g. a customer impacting issue) we can deploy quickly and safely.

> Deployment != Release

Changes can be deployed without releasing them to the customer using techniques such as [Feature Toggles (aka Feature Flags)](https://martinfowler.com/articles/feature-toggles.html).

