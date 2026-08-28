---
title: "Why Data Engineering Matters for Managers and Business Owners — Part 3"
date: 2026-08-17
description: "In the transition to AI, what new challenges have sprung up for data teams, and how can you manage them without falling into the technical-debt trap?"
tags: ["data-engineering", "business", "ai", "technical-debt", "team"]
categories: ["Data Engineering"]
author: "Mostafa Ghadimi"
showToc: true
---

About ten months have passed since my last post, and in those ten months many of the day-to-day processes we deal with have changed — and it looks like the paradigms are shifting completely. As AI models get more capable, the amount of code I personally write as a data engineer has dropped sharply (trending toward zero), and I lean more on tools to reach my goal.

Inside companies and businesses this paradigm shift happens on a slower cycle, and there's still not much confidence in the outcome. In particular, the claim that a large share of the team can be replaced by AI has no solid backing yet (at least at the time of writing). This shift has both solved some old challenges and created new ones — which I'll try to cover here, along with their solutions.

---

## New challenges of the transition to AI

### 1. Fading project ownership

When a tool writes much of the code, the sense of ownership over the project fades. Nobody sees themselves as the owner of the architecture and the quality of the output, so when something goes wrong, responsibility for it hangs in the air.

> **Solution:** move ownership from the "code" level to the "design and decision" level. The data engineer should own the architecture, the choices, and the review of the output — even if they didn't write every line of code themselves. The tool writes the code, but the responsibility stays with the human.

### 2. The high speed of regenerating and offering diverse solutions

AI can put several different solutions in front of you in a short time — and naturally that speed is far greater than ours. That tempts us to drown in options and grab the first thing that works instead of making a deliberate choice.

> **Solution:** a variety of solutions is an opportunity, not a threat — as long as we have selection criteria. Before generating solutions, define the decision criteria (maintenance cost, readability, scalability) so that AI's speed serves the right decision rather than replacing it.

### 3. Unrealistic expectations of data-team speed, and technical debt

When everyone thinks AI solves everything instantly, unrealistic speed is expected of data engineering teams too. The result is accumulating technical debt.

A more complex stack, missing ownership, and a growing context eventually keep the project from reaching the desired result. Not budgeting proper time for sound design and implementation of the infrastructure lowers development cost in the short term, but in the long term it brings staggering maintenance costs and an unreliable project.

> **Solution:** manage expectations transparently. Don't equate prototype speed with the speed of a reliable production product, and set aside a defined, defensible amount of time for infrastructure design. Technical debt you don't keep an eye on eventually shows up at a much higher cost.

### 4. Exposing sensitive data

Handing work to models means the users' and the company's sensitive data may be exposed to them. The risk of data leakage is no longer a hypothetical concern.

> **Solution:** first of all, define the data boundary. Sensitive data (PII) must be masked or removed, and for sensitive work reach for self-hosted models or services that guarantee your data won't be used for training. Treat a model's access to data like an external contractor's access — limited and audited.

### 5. High cost, low return

Many companies that chase being AI-First instead of Data-First ignore the principles of data engineering and infrastructure just to reach a result quickly. What comes out are temporary solutions; heavy dependence on AI and never reviewing the stack create anarchy across the whole system. We have no good estimate of project runtimes, and the side effects aren't measurable either.

> **Solution:** put AI on top of a sound data foundation, not in place of it. Infrastructure and fundamentals first, speed second. A temporary solution that isn't reviewed and documented becomes the main cost itself.

---

To sum up: even right now we're already handing our projects to AI, but we neither fully trust it — because of the data-leakage risk — nor does this tool alone meet all our needs.

So the real question isn't "should we use AI or not" — the question is how to put it on top of a sound data infrastructure so that its speed doesn't turn into technical debt.

How do you manage this paradigm shift on your team?

---

💬 I also published this piece on [LinkedIn](https://www.linkedin.com/posts/mostafaghadimi_%D8%A7%D9%87%D9%85%DB%8C%D8%AA-%D9%85%D9%87%D9%86%D8%AF%D8%B3%DB%8C-%D8%AF%D8%A7%D8%AF%D9%87-%D8%A8%D8%B1%D8%A7%DB%8C-%D9%85%D8%AF%DB%8C%D8%B1%D8%A7%D9%86-%D9%88-%D8%B5%D8%A7%D8%AD%D8%A8%D8%A7%D9%86-%DA%A9%D8%B3%D8%A8%D9%88%DA%A9%D8%A7%D8%B1-activity-7495104349753352192-FPYh); I'd be glad to continue the discussion there.
