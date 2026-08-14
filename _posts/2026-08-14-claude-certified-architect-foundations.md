---
layout: post
title: "How I Prepared for and Passed the Anthropic Claude Certified Architect Foundations Exam"
subtitle: "What the exam actually tests, which resources helped me most, and what I would do differently next time."
tags: [AI, Claude Code, Certification]
comments: false
social-share: true
---

*Written August 2026, based on Exam Guide version 1.0.*

I recently prepared for and passed the **Claude Certified Architect – Foundations** exam after five weeks preparing alongside my full-time job. That number needs context: I had already been using Claude Code in an enterprise engineering environment for close to a year, including configuring MCP servers used in production and building agents. With that experience, I believe two focused weeks would be enough for an experienced practitioner.

That experience transferred more than I expected. The exam scenarios are fundamentally trade-off problems: reliability versus simplicity, isolation versus shared context, capability versus blast radius, latency versus cost. The underlying reasoning is familiar to anyone who has designed non-trivial software systems.

My biggest takeaway: this is not primarily a "Claude API or prompt engineering knowledge" exam. It is an architecture and decision-making exam built around realistic production scenarios.

## The exam at a glance

Before getting into preparation, here is the format as published in the official Exam Guide.

- **Questions**: 60, multiple choice, one correct answer each
- **Time**: 120 minutes
- **Scenarios**: 4 selected from a pool of 6, each providing the narrative context for roughly a quarter of the questions
- **Domains**: 5, unevenly weighted
- **Passing score**: 720 on a scaled range of 100–1,000, which is a scaled score and not a percentage of questions answered correctly
- **Delivery**: Pearson VUE, online proctored or at a test centre
- **Fee**: $125 USD
- **Credential validity**: 12 months
- **Target candidate**: roughly six months of practical experience across the Claude APIs, the Agent SDK, Claude Code and MCP, rather than someone who has only completed tutorials

One point worth checking before you invest weeks of preparation: eligibility. As of August 2026, the Claude Certification Program is open to organizations in the Claude Partner Network, and passing counts toward Claude Partner Network standing — registration goes through your employer if it's a partner, using your professional email.

The certification page is public, though, and only the registration itself is gated. Anyone can read the domain weightings, the format, and — more importantly — download the Exam Guide from there. If your organization isn't in the network, the exam is out of reach for now, but the preparation described below isn't.

The five scored domains, in the order the guide presents them:

1. Agentic Architecture & Orchestration — 27%
2. Tool Design & MCP Integration — 18%
3. Claude Code Configuration & Workflows — 20%
4. Prompt Engineering & Structured Output — 20%
5. Context Management & Reliability — 15%

The important thing is not to memorize these numbers. It is to understand how those areas interact inside a production scenario — and to notice that a single domain carries more than a quarter of the exam.

## What does the certification actually test?

A common misconception before starting the preparation is that this exam is mainly a test of Claude API, or Claude Code knowledge. It isn't.

The questions are grounded in realistic customer situations — customer-support agents, multi-agent research systems, Claude Code development workflows, CI/CD, developer productivity and structured data extraction — and you are expected to make decisions about architecture, configuration and production trade-offs. The wording reflects that: you are not asked whether a feature exists, you are given a situation, a set of constraints and several technically plausible options, and asked which design is the best fit.

That distinction matters because several answers may work in the real world. The correct answer is the one that addresses the actual problem while respecting all the constraints in the scenario. The official guide is explicit on this point: the incorrect options are written to look reasonable to someone with incomplete knowledge or experience. For example, you are not simply asked:

> "What does feature X do?"

You are much more likely to encounter something closer to:

> "Your production agent behaves incorrectly under these conditions. Which architectural change would most effectively address the problem?"

## The architectural traps I underestimated

This is the part I would spend more time on if I were preparing again.

**Batching is not a cost optimization.** I had filed the Message Batches API under "cheaper" and left it there. The question that actually decides is whether the workflow blocks: a pre-merge check that developers are waiting on needs predictable latency, an overnight technical-debt report does not. Once I started reading scenarios for blocking behaviour rather than for cost, that family of questions stopped being ambiguous.

**MCP resources versus tools.** In the servers I had built, almost everything was exposed as a tool; tools are familiar and easy to invoke. When an agent has to call a tool simply to discover what content exists, you are paying turns for exploration. Content the agent needs to read belongs as a resource; actions that change state belong as tools. Applying that split back at work reduced the exploratory chatter noticeably, which is how I know the distinction is not academic.

**Context is a budget, not a container.** My instinct on long-running work had been to keep everything in one conversation and compact it when it got heavy, or systematically start a new session. The stronger discipline is to spend context deliberately: trim verbose tool output, preserve structured facts rather than raw transcript, isolate subagent context so exploration never touches the main thread, and persist state when the work crosses context boundaries.

**Schema design beats prompt instructions.** The general reflex to improve a prompt is to add another sentence to it: be careful, do not invent, say when you do not know. Designing the output schema so that uncertainty is representable is stronger: nullable fields, appropriate enums, an explicit unknown case. A model that has a valid way to say the information was missing stops filling the gap with something plausible.

**Deterministic guarantees versus probabilistic compliance.** This is the pattern behind all the others. If a business rule must always hold, a hook or a programmatic prerequisite is the answer, not another instruction in the system prompt. Any option that asks the model to be reliable about something the system could simply enforce is usually the option to eliminate.

Production experience helped me most here — and it is also where it can mislead you. The architecture you already run may work perfectly well; the exam asks which pattern is most appropriate under the constraints provided. For example, when a customer explicitly demands a human for a simple password reset, my production instinct — optimized for ticket deflection — was to try resolving it autonomously first. However, Anthropic strictly requires honoring explicit customer requests for human agents immediately, without attempting investigation. Cost-saving enterprise habits can easily point you to the wrong answer.

## What I actually used to prepare

My preparation was deliberately hands-on.

**Learn:** I worked through the free Anthropic Academy courses, especially *Building with the Claude API* and *Claude Code in Action*. The first covers tool use, structured output, MCP and agent workflows; the second focuses on configuration, Plan Mode, skills, hooks, automation and longer-running workflows. I did not simply watch them.

**Build:** every concept reproduced in code rather than watched — agent loops, Claude Code configuration, MCP tools, a structured extraction pipeline, a small multi-agent system. The exam rewards operational understanding over recognition, and the gap between the two only shows up under a scenario.

**Sketch:** a tablet with me throughout, redrawing each pattern by hand before assuming I had understood it. Drawing an orchestrator delegating to three workers, then the same task as a parallel pipeline, is what made the difference obvious: the topology is nearly identical, and only the moment the subtasks are decided separates them.

One thing I would change, however, is the order. I started with the courses and only properly studied the Exam Guide afterwards. I would do the opposite: **read the Exam Guide first.**

The Exam Guide is the map: domains, task statements, scenarios and sample questions. Read it before starting the courses, then turn each objective into a simple checklist:

Can I explain it? Can I implement it? Can I recognize the wrong architectural choice in a scenario?

Every "no" becomes a study topic. That changed the way I used the courses; instead of wondering whether a concept was important, I knew exactly which objective it was helping me cover.

## My preparation loop

My actual process became:

**Exam Guide → Courses → Hands-on practice → Mock exam → Analyze mistakes → Documentation → Repeat**

I generated mock exams using Gemini Pro, with the Exam Guide as the source of truth. There is no full official mock exam, although the guide includes sample questions and explanations. The idea was not to generate random trivia: I wanted questions that reproduced the exam's reasoning style — realistic scenarios, explicit constraints, four plausible options and one best answer.

```
Using only the exam objectives provided below, generate one
scenario-based multiple-choice question.

Requirements:
- Create a realistic production architecture scenario
- Include explicit constraints such as latency, cost,
  reliability, security, or context limits
- Provide 4 technically plausible options
- Have exactly one best answer given the stated constraints
- After I answer, explain why each option is correct or incorrect
- For every incorrect option, identify the constraint or
  architectural principle it violates

Do not use information outside the provided objectives.
Do not reproduce real exam questions.
```

This became one of the most useful parts of my preparation, because I could generate new questions the moment I discovered a weak area.

But there is one important rule: generated answers are practice, not truth. My source hierarchy was the Exam Guide, then Anthropic's documentation, then the official courses, then my own experience, and only then generated material. Whenever a mock explanation felt questionable, I went back up that ladder.

## Advice on third-party practice exams

For external material, I took one practice exam by Frank Kane on Udemy — *Anthropic Claude Certified Architect – Full Practice Exams* — which was a useful way to exercise the reasoning under a different question style.

Beyond that, I did not find anything genuinely useful online as of July 2026. If you know of a good resource, please leave it in the comments.

Whatever you use, don't try to maximize the number of mock exams. The value comes from what happens after the question: for every wrong answer, go back to the course or the documentation, name exactly what you had misunderstood, then find another question targeting the same concept.

## How I approached difficult questions

My default strategy was elimination. When I did not immediately know the answer, I stopped asking "which option is correct?" and started asking "which options can I eliminate?"

For every choice, I checked:

- Does it solve the actual problem?
- Does it respect every explicit constraint?
- Does it introduce infrastructure that the scenario does not require?
- Does it give an agent more tools or permissions than necessary?
- Does it rely on probabilistic behavior where deterministic enforcement is required?

Usually two options become easy to eliminate, and what remains is a much smaller architectural trade-off. My mental model during the exam became:

**Scenario → Objective → Constraints → Eliminate → Compare trade-offs → Choose**

The trap is selecting an answer because it would work. Working is not enough, the question is asking for the design the scenario actually demands.

## Time management

The 120-minute limit gives you an average of two minutes per question, but I would not treat two minutes as a target. Some questions take seconds when you know the underlying concept; others require several minutes because you need to read the scenario carefully and compare architectural trade-offs.

I therefore moved quickly through the obvious questions and preserved mental bandwidth for the long scenarios. A slow question does not mean you are doing badly, it usually means the question is testing judgment rather than recall.

## The mistakes I would avoid

- Starting with the courses instead of the Exam Guide.
- Watching lessons instead of reproducing them in code, depending on your experience building with Claude.
- Trusting generated or third-party answers without checking the documentation.
- Memorizing mock answers instead of understanding why the alternatives fail.
- Assuming that what you already ship in production is automatically the pattern the exam expects.
- Learning new material the day before. I used the final day to reread the Exam Guide and consolidate, not to expand the syllabus.

## What I took away from the certification

The most valuable outcome was not the credential itself. Preparing forced me to put explicit names and boundaries around patterns I was already using: agent orchestration, tool isolation, structured handoffs, context management, deterministic enforcement and human escalation.

That made the preparation useful beyond the exam. You stop thinking about agents as a collection of prompts and tools, and start reasoning about them as software systems with state, failure modes, interfaces, permissions and architectural trade-offs. That, ultimately, is what I think the certification is testing:

Not whether you can make Claude do something. Whether you can design a system around Claude that behaves appropriately when the constraints become real.

And it travels. The exam is written against Anthropic's stack, but most of what it drills is not vendor-specific: when to let a model decide and when to enforce a rule in code, how to split a task across agents and when not to, what belongs in the context window and what belongs in a file, how to design a schema that can represent what the model doesn't know. Those decisions look the same whether the model underneath is Claude, Gemini or GPT. The architectural reasoning transfers, and that is the part that keeps its value when the stack changes under you.

---

Good luck to everyone preparing for the Claude Certified Architect – Foundations exam. And if you have already taken it, I would be interested to hear which part of the preparation you found most challenging.
