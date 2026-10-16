# Chapter 19: Feedback Loops and What You Owe When One Breaks

**Summary**: *Two subjects meet in this chapter and they turn out to be the same subject. Managing what stakeholders expect is about converting an unachievable demand into a measurable target somebody negotiated. Designing feedback is about making sure a failure produces a signal that arrives at a specific person. Both fail the same way: something everyone assumed was happening, that nothing in the system actually does.*

## Give a Bounded Choice, Not a Decision Taken for Them

When a stakeholder's expectation cannot be met as stated, there is a move that separates the strong answers from every alternative.

Put a **bounded choice** in front of them with the **trade-off visible**.

That rules out both directions of the same failure. Deciding on their behalf to accept the impossible target is one. Deciding on their behalf to abandon it is the other. Both **take a decision that belongs to somebody else**, and both do it **silently**.

The same shape appears in estimation. Show the **estimate** and the **assumptions** behind it as **separate things**, so a client can see which assumptions are carrying the figure and which ones, if they moved, would move it. A single figure with the judgement buried inside it cannot be argued with, which sounds like an advantage and is not: the client's only available responses are to accept it or to distrust it.

## Reframe the Requirement, Do Not Accept or Refuse It

The recurring scenario is a stated bar the system cannot reach. Perfect accuracy. Zero errors. Never wrong on anything that matters.

Four responses appear, and three are failures. **Accepting the bar** commits to something that will not happen. **Abandoning the engagement** treats an unrealistic opening position as final. **Adjusting a system parameter**, a temperature, a threshold, a model tier, changes the system while leaving the expectation exactly where it was, which is the most seductive wrong answer because it feels like progress.

What works is **rewriting the requirement into two pieces**: **criteria somebody can actually measure**, and **error handling sized to what each mistake costs**.

Measurable criteria, because "accurate" is not a target and "94% on this evaluation set" is. Risk-proportionate handling, because the honest answer to "what about the errors?" is not that there will be none, it is that different errors get different treatment according to what they cost.

Two related versions of the same move are worth knowing.

**Renegotiate the target against measured evidence**. Bring three things: what the system measurably does, how well the people currently doing it perform, and what each kind of mistake costs. A **target argued against a measurement holds**; a target argued against an **opinion does not**.

**Preserve capability, honesty and felt experience together**. Those three are what the wrong answers each sacrifice: gutting the feature protects honesty and destroys capability, over-promising protects the felt experience and destroys honesty, and quietly staging the rollout protects capability while misleading everybody.

One factual correction that appears as a trap: **response time is not somebody else's property**. Which tier you run, how long the prompt is, whether you stream, whether you batch, how retrieval is built, every one of those shifts it. An option that treats response time as an immovable property of the platform is wrong on the facts.

## Two Practices Manage Scope Change

Scope change is handled at exactly two moments, and a scenario asking for two practices is asking for these.

When a **request arrives: expose its cost before anything is promised**. Not after the work starts. Not in the next status report. The moment somebody asks, the price is visible.

When a **request is accepted: reset the agreed plan**. The dates, the scope, the commitments all move together, in writing.

Every alternative conceals the price at one moment or the other. Absorbing the change quietly conceals it at the first. Accepting it and leaving the plan alone conceals it at the second, which is worse, because now the plan is a document everybody knows is false and nobody has said so.

## A Defensible SLA Is Measurable and Remediable

The rule is short and it settles every SLA item.

Whoever **makes the promise** has to be able to **measure** it, and has to be able to **do something about it when it slips**.

Measurable, because a commitment nobody can evaluate is not a commitment. Remediable, because a commitment you cannot act on when it is missed is a wish.

That admits one kind of quality clause and excludes the popular ones:

| Clause | Verdict |
|---|---|
| A **quality number on an agreed evaluation set** | **Holds**. Measurable, and you can act on it |
| An absolute promise of correctness | Fails. Not achievable, therefore not remediable |
| A promise about which model is used | Fails. It is an implementation detail, and it is a promise about a mechanism rather than about an outcome the client cares about |
| A promise about how autonomously the system operates | Fails for the same reason |

Then the part people forget, which is the whole point of the exercise: an **SLA states what you owe** when the **number is missed**. A commitment with no remedy attached is a statement of intent. The remedy is what makes it an agreement.

And on accuracy specifically: turn it into a **target both sides agreed**, judged against **how well people currently do the work** and against **what each kind of mistake costs**, rather than hiding the measured figure, sitting on it, or promising past it. Concealment and delay are the two failures that feel survivable at the time and are not.

## Four Properties That Outlive a Feedback Session

An SLA is agreed once. Agreement is **maintained in a recurring session**, and whether that session earns the hour it costs is decided by four properties, every one of which is about the time on either side of the meeting rather than the meeting itself.

**Artifacts circulated in advance**, so the hour is spent on the disagreements rather than on people reading in silence.

**A focused agenda**, so attendees can prepare against something and so the session has an end condition other than the clock.

**Decisions and follow-ups written down**, because a decision nobody recorded gets retaken next month by whoever remembers it differently, and both parties will remember it in their own favour without anybody lying.

**A named owner and a date** on every follow-up, which is the only one of the four that reliably survives contact with a busy quarter.

Test any proposed format by asking what remains a week later. A session that produced **no artifact anybody can point at** produced **goodwill**, which is worth having and is **not agreement**. That is also the fast way to rank the options in a scenario: the ones that improve the meeting as an experience change how the hour feels, and the ones on the list above change what exists after it.

## A Report Is Wrong When It Measures the Wrong Thing

When stakeholders are dissatisfied with reporting, there is one diagnosis that matters and three remedies that do not.

What the **report counts** and **what its readers answer for** have **come apart**. Fix the content.

Sending it more often does not help. Sending more of it does not help. Adding another report does not help. All three change the **frequency**, the **volume** or the **existence of reporting** while **measuring the same things**, and the audience is still looking for a number that answers for what they are answerable for.

## The Signal That Reaches a Person

Now the mechanical half, and the documented behaviour that makes it sharp.

Name it: the **signal that reaches a person** is the only part of a feedback loop that counts. Everything upstream is plumbing, and plumbing has a documented tendency to lose things quietly.

Webhook delivery is the clearest published example, and every one of its properties is a way a loop can be open without anybody knowing.<sup>[1]</sup>

- **Events** go only to **endpoints subscribed at the moment the event is emitted**. An event emitted while nothing is subscribed is **never delivered**, and **subscribing later does not backfill** it.
- Each **endpoint and event** gets up to **three delivery attempts**, with jittered exponential backoff between **5 and 120 seconds**. After that, the event is **dropped rather than queued**, with **no signal that it was lost**.
- An **endpoint is automatically disabled**, with a machine-readable reason, after a redirect response, an invalid address, or sustained delivery failures. For that last case the trigger is the **uninterrupted duration of failure** rather than a **count of deliveries**, and a **single successful response resets the window**.
- **Events emitted while an endpoint is disabled** are **not replayed** when it is **re-enabled**.
- **Ordering is not guaranteed**.

Two design rules fall straight out. **Drive state from the fetched resource** rather than from the **event payload**, because ordering is not guaranteed and events go missing. And where **every transition must be observed**, **reconcile by listing or fetching** through the API, because notification alone cannot promise you saw everything.

The **named anti-pattern** is the **loop that reports its own health by staying quiet**. A dropped event, a disabled endpoint and a healthy system all look identical from the receiving end.

## Two More Places a Signal Is Owed

**Spend limits** carry a small governance loop with documented behaviour.<sup>[2]</sup> The API returns each member's effective limit together with the **scope level the limit was resolved** from and their period-to-date spend. **Increase requests are raised by members** rather than **through the API**, and resolved through explicit approve and deny endpoints, with **both outcomes terminal** and at **most one request pending per member**.

Two details are worth carrying. A **denied request hides the member's request control** for **30 days from resolution**, while an **administrator may still raise the limit** directly at any time. And Anthropic **emails the member** on **approval or denial** by default, with a documented suppression parameter for organisations that notify through their own system, which matters because **two notifications about the same decision** is **worse than one**.

One measurement caveat sits in the same place: **period-to-date spend** is **informational rather than transactional** and may **read as zero** when the reading is **temporarily unavailable**. A control built to trigger on zero spend will trigger on a missing reading.

**Model retirement** carries its own notification loop.<sup>[3]</sup> **Impacted customers are always notified** by **email** and in the **documentation**, and Anthropic notifies customers with **active deployments on models that have upcoming retirements**. And the part that puts the work back on you: customers can access an **audit of their own API usage** in order to **locate any application still calling a deprecated model**. The notification tells you a model is going. Only the usage audit tells you which of your systems still depends on it.

## How Wrong Answers Are Built on Expectation and Feedback Questions
- A **decision taken on the stakeholder's behalf**. In either direction. The test: does the stakeholder get a **choice, with the trade-off visible**?
- A **system change offered against an expectation problem**. The test: after this, does the stakeholder **expect anything different**?
- A **commitment nobody can measure or act** on. The test: how is this **evaluated**, and what is **owed when it is missed**?
- **Reporting changed in frequency instead of in content**. The test: does the report now measure what the **audience answers** for?
- A **notification treated as a guarantee of delivery**. The test: what happens to this signal if the **endpoint is down**, or **nothing is subscribed**?

| When the scenario says | The answer is usually | The trap is usually |
|---|---|---|
| A stakeholder expectation cannot be met | Offer a bounded choice with the trade-off visible | Accepting the bar, or abandoning it, both decided for them |
| An unachievable accuracy bar is stated | Rewrite it as measurable criteria plus error handling sized to each mistake | Tuning a parameter, which leaves the expectation untouched |
| A target must be revisited | Renegotiate against measured behaviour and how well people currently do it | Arguing from opinion, or unilateral engineering |
| An estimate is challenged | Show the figure and its assumptions as separate things | A single figure with the judgement buried inside |
| Latency is called an immovable constraint | The tier, how much prompt you send, streaming, batching and how retrieval is built all shift it | Accepting it as a platform property |
| Scope change must be managed, two practices wanted | Expose the cost when the request arrives, and reset the plan when it is accepted | Absorbing it quietly, or accepting it without moving the plan |
| An SLA clause is proposed | A quality number on an agreed evaluation set, with a stated remedy | Absolute correctness, or promises about model choice or autonomy |
| Stakeholders are unhappy with reporting | Change what the report counts | Reporting more often, in more volume, or adding another report |
| Every state transition must be observed | Reconcile by listing or fetching through the API | Relying on notifications, which drop events silently |
| A webhook endpoint was down | Events are not replayed on re-enabling | Assuming a queue held them |
| Nothing was subscribed when an event fired | It was never delivered and will not backfill | Subscribing afterwards and expecting history |
| A member's spend request was denied | Their request control hides for 30 days; an administrator can still raise it directly | Treating denial as permanent for everyone |
| Period-to-date spend reads zero | It may be an unavailable reading, not real usage | Triggering a control on it |
| A model is being retired | Audit your own API usage to find what still calls it | Relying on the notification to identify your systems |
| A recurring feedback session has to be designed | Artifacts in advance, a focused agenda, written decisions and follow-ups, a named owner with a date | Anything that improves the meeting itself and leaves nothing behind it |

The habit worth carrying: for every signal in your design, ask **what happens to it when nobody is listening**. If the answer is silence, you have built a loop that reports success by saying nothing.

## What to Remember
- When an expectation cannot be met, give the stakeholder a **bounded choice with the trade-off visible**. Accepting the impossible target and abandoning it are the **same failure in two directions**: both **decide something that was theirs to decide**.
- Rewrite an unachievable requirement into **two pieces**: **criteria somebody can measure**, and **error handling sized to what each mistake costs**. Adjusting a temperature, a threshold or a model tier **changes the system** and **leaves the expectation exactly** where it was.
- Renegotiate targets against what the **system measurably** does, how **well people currently do that work**, and what **each kind of mistake costs**. A target argued against a **measurement** holds; one argued against an **opinion** does not.
- Preserve **capability, honesty and felt experience** together. Gutting the feature, over-promising, and quietly staging the rollout **each sacrifice one of the three**.
- Response time is **not somebody else's property**: which **tier you run**, how **much prompt you send**, **whether output streams**, **whether work is batched**, and how **retrieval is built** all shift it.
- Scope change is handled at **two moments**: **show the price when the request arrives**, before anything is promised, and **reset the agreed plan when it is accepted**. Every alternative **conceals the price** at one moment or the other.
- Show an **estimate and its assumptions as separate things**, so a client can see which **assumptions carry the figure** and which would **move** it.
- Whoever makes an SLA promise must be **able to measure** it and **able to act when it slips**. A **quality number on an agreed evaluation set** holds; absolute correctness, model choice and autonomy do not. An SLA states what you **owe when the number is missed**, and a commitment with **no remedy** is a **statement of intent**.
- When reporting dissatisfies its readers, what it **counts** and what they **answer** for have **come apart**. Frequency, volume and adding another report all **leave the measurement unchanged**.
- Webhook events reach **only endpoints subscribed at the moment of emission**, are **never backfilled**, get **three delivery attempts** with backoff between **5 and 120 seconds**, and are then **dropped rather than queued** with **no signal of loss**.
- An endpoint is **auto-disabled** after a **redirect**, an **invalid address**, or **sustained failure**, where sustained means **uninterrupted duration rather than a delivery count**, and **one success resets the window**. **Events emitted while disabled are not replayed**, and **ordering is not guaranteed**.
- **Drive state from the fetched resource**, and **reconcile by listing or fetching through the API** wherever every transition must be observed. **Notification alone** cannot promise you saw everything.
- Spend limit responses include the **scope level the limit resolved** from. **Members raise increase requests, not the API**; approve and deny are **both terminal**, **one pending request per member**, and a denial **hides the member's request control for 30 days** while an **administrator can still raise the limit directly**. Emails go out by default with a **documented suppression parameter**. **Period-to-date spend is informational** and may **read zero when unavailable**.
- Retirement notices always go by **email and into the documentation**, to customers with **active deployments on affected models**, and an **audit of your own API usage** is what identifies which **applications still call a deprecated model**.
- A feedback session is worth the hour when **four things outlive it**: **artifacts circulated in advance**, a **focused agenda**, **decisions and follow-ups in writing**, and a **named owner with a date**. A session leaving **no artifact anybody can point at** has produced **goodwill rather than agreement**.

## Endnotes for this chapter

1. Webhook events are delivered only to endpoints subscribed to that event type at the moment the event is emitted, an event emitted while no endpoint is subscribed is never delivered and subscribing later does not backfill it; each endpoint and event receives up to three delivery attempts with jittered exponential backoff between 5 and 120 seconds, after which the event is dropped rather than queued, with no signal that it was lost; an endpoint is automatically disabled with a machine-readable reason after a redirect response, an invalid address, or sustained delivery failures, where the trigger for the last is the uninterrupted duration of failure rather than a delivery count and a single successful response resets the window; events emitted while an endpoint is disabled are not replayed on re-enabling; and ordering is not guaranteed, so state should be driven from the fetched resource, with reconciliation by listing or fetching through the API where every transition must be observed. Anthropic, *Subscribe to webhooks* (Claude Platform Docs, accessed 2026-08-25), [https://platform.claude.com/docs/en/managed-agents/webhooks](https://platform.claude.com/docs/en/managed-agents/webhooks)

2. The Spend Limits API returns each member's effective spend limit together with a field identifying the scope level the limit was resolved from and the member's period-to-date spend; spend limit increase requests are raised by members rather than through the API and are resolved through explicit approve and deny endpoints, with both outcomes terminal and at most one request pending per member; a denied request causes the member's request control to be hidden for 30 days from resolution while an administrator may still raise the limit directly at any time; Anthropic emails the member on approval or denial by default, with a suppression parameter documented for cases where the organisation notifies the member through its own system; and period-to-date spend is documented as informational rather than transactional and may read as zero when the spend reading is temporarily unavailable. Anthropic, *Spend Limits API* (Claude Platform Docs, accessed 2026-08-25), [https://platform.claude.com/docs/en/manage-claude/spend-limits-api](https://platform.claude.com/docs/en/manage-claude/spend-limits-api)

3. Impacted customers are always notified by email and in the documentation, Anthropic notifies customers with active deployments on models that have upcoming retirements, and customers can access an audit of their own API usage in order to locate any application still calling a deprecated model. Anthropic, *Model deprecations* (Claude Platform Docs, accessed 2026-08-25), [https://platform.claude.com/docs/en/about-claude/model-deprecations](https://platform.claude.com/docs/en/about-claude/model-deprecations)
