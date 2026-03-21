---
title: "The Event Log Is the Foundation of the Agent Economy"
date: 2026-03-20
draft: false
tags: ["kafka", "agents", "event-driven", "data-engineering", "streaming"]
description: "The event log is the natural substrate for agent-to-agent communication. Most of what we currently call the modern data stack is a transitional architecture that exists to solve a problem agents won't have."
author: "Neil Turner"
ShowToc: true
TocOpen: false
---

It might be a coincidence that I’ve been getting pinged about Kafka by recruiters a lot lately, or it might be that companies are starting to think about its applications more broadly in the context of whatever shape the “Agent Economy” takes.  Either way, I couldn’t help but notice that the universe seems to be prodding me in that direction, and it got me thinking deeply about what the atomic units of systems design look like in that world.  I’ve already been talking about what happens when the agentic paradigm strips away the need for SaaS entirely with colleagues and friends, and how companies are supposed to stay relevant in that world.  But my passion has always been “Big Data” in whatever form that practice has taken throughout the decades of my career, so my pondering turned naturally to what this sea change looks like in the world of Data Engineering.

## The data stack is basically duct tape

Most of what Data Engineering does is fix the problem that application siloing introduced.  When Salesforce needs to agree with Netlify and half a dozen other spreadsheets and vendor applications that are floating around an organization, so that someone on the revenue team can answer a specific question, that is where Data Engineers earn their paychecks.  The entire Fivetran-into-Snowflake-into-dbt pipeline that we've collectively spent billions on exists to reunify the silos of data that SaaS applications create, so that humans can reconcile things for business purposes.  

That's not to say that these tools are bad…  they're not.  They're great at what they do.  But they exist to solve a problem that might not exist in the future we're headed towards.  

Applications don't share data natively, so we needed tools that could do it after the fact.  Applications are built with rigid, pre-defined integration points. Salesforce exposes an API, but it's Salesforce's API, with Salesforce's schema, on Salesforce's schedule. The integration surface is fixed at design time by the vendor, not negotiated at runtime by the systems that need data from each other. That rigidity is why we need Data Engineers to come help stitch everything back together in order to derive cooperative outcomes.

Agents are different in a specific way that matters here: they can dynamically negotiate data exchange because they understand context.  Agents can reason about their business domain, conclude they need information from X Y and Z topics, subscribe to them, build software to parse the results, aggregate/join, and ultimately, act on their directives.  They don't need a centralized hub where all data accumulates in order for them to stitch things together.  The silo reunification problem doesn't vanish overnight, but the assumption that you need a centralized pipeline to solve it starts to weaken. 

So what happens when AI agents start encroaching on the SaaS application layer?  It'll start to happen soon, in one form or another.  [Bain's 2025 Technology Report](https://www.bain.com/insights/will-agentic-ai-disrupt-saas-technology-report-2025/) lays out concrete scenarios where agentic AI disrupts the SaaS revenue model. [Deloitte's 2026 TMT predictions](https://www.deloitte.com/us/en/insights/industry/technology/technology-media-and-telecom-predictions/2026/saas-ai-agents.html) explore how agents begin absorbing the workflow automation that SaaS products traditionally provide. The question these analysts are asking is whether agents kill SaaS. The question I'm more interested in is what happens to the data infrastructure underneath it when they do.

The way I see it, a few things will happen.  First, the event log survives because the moment you let autonomous systems make consequential decisions across organizational boundaries, you've traded one problem for another. You no longer need infrastructure to reunify data. You need infrastructure to prove what happened.

Second, the data warehouse survives, but not as the central element in the ecosystem.  It becomes the primary derived view… the biggest, most comprehensive materialized view of the whole system.  The one that keeps everything and indexes it for ad-hoc query, but a derived store, not a primary one. The event log is the authority, and the warehouse is one of many projections over it.  The warehouse allows us to train the agents, perform exploration, and comply with audits.  It never gets retired because regulatory compliance and exploratory historical analysis are permanent fixtures of how we do business (for now).    

And third, the Fivetran/dbt/pipeline layer is the layer that atrophies, because its access pattern was "pull from SaaS APIs on a schedule and land it somewhere humans can query." If agents are negotiating data exchange dynamically, that scheduled pull-and-land pattern loses its reason to exist.

The new data stack isn't organized around 'SaaS apps don't talk to each other'.  It becomes two primary layers and a projection layer. Agents and event logs are analogous to compute and storage.  Derived stores/dynamic projections are analogous to memory.  The ETL layer fades because its original problem is dissolving.  

![Current vs. agent-era data stack](/images/data-stack-before-and-after.png)

## The log is demanded by the operational reality

When autonomous data agents start making consequential business decisions at the data foundation level (i.e., after SaaS transactions), you need an immutable, ordered, replayable record of every decision and state mutation (both for business reasons and technological reasons). Agents don't care what they write to… they'll happily write to databases, file systems, and APIs. The reason you need the log is governance. The alternative to a durable audit trail is trusting an autonomous system with no way to replay what it did or why. In any regulated industry, and increasingly in any enterprise context, that's unacceptable.

It's not a stretch to see why this continues to be important.  We all know that agents make stuff up sometimes, so we are going to need durable logs that record these decisions, in addition to the raw data passing between the components of an org.  The event log is the only data structure that gives you the ability to completely replay a sequence of events that led up to a decision.  This will continue to be vital for compliance and auditing purposes, because even if agents eventually develop long-term memory, an agent's self-reported history is not an independent audit trail. The log exists because you need a record the agent can't edit (and this should be disabled at the platform level).  

None of the current thinking that I'm aware of in the A2A protocol space aligns perfectly with this as of today, but from where I'm sitting it's the only logical choice to solve the problem of tracing an agent's behavior in a large distributed A2A system. In order to guarantee auditability, you need ordering guarantees, durability, and replayability, and no other data structure provides all three as naturally as the distributed log. Most agentic systems today record agent actions to Postgres, which works fine for a single application with a handful of agents. It won't scale easily to hundreds or thousands of collaborative agents all working together to churn through data from humans, applications, and other agents. Kafka, on the other hand, handles this kind of write volume without breaking a sweat. 

## The event topology self-organizes

Ever seen those slime molds, particularly *Physarum polycephalum*, that can organize themselves into the most optimal network topology over time without relying on any form of centralized control?  I think that's what we get when we tell this system to regulate itself and find the most efficient substrate configuration for information transfer, assuming some mechanism for pruning stale topics and brokers exists (as it must).  In this world, standing up a Kafka cluster is simply propagating the "food" (data) to the rest of the "organism" (other agents).  This happens organically as agents decide they need access to certain kinds of information in order to derive value from various different projections over the available data supply. Obviously, there needs to be a governance layer to prevent garbage from accumulating and topic overlap, but over time the system converges on good coverage of the domain by providing the most valuable combinations of raw materials.   

The first version of this idea is unglamorous.  A fraud detection agent that needs to cross-reference sales events with operational events creates a new topic, subscribes to the relevant source topics, and starts emitting derived events into its own namespace. Agents creating their own clusters and topics over admin APIs is already technically feasible today.  With adequate guardrails and governance, it can proceed on its own from there. No Jira ticket. No platform team review.

It's important to emphasize that agents will never process first-order domain events themselves.  They're way too slow and that would be cost prohibitive.  Instead, they will create their own consumers that can aggregate, join, and compute over the raw data in whatever way is needed to allow them to orchestrate and make decisions.  This extends to how agents build their own knowledge as well. A vector store for semantic retrieval, a structured table for key-value lookups, a text search index for discovery… These are all just materialized views over the same processed log data, optimized for different query patterns. The agent curates these projections for itself based on what it needs to make decisions. If one becomes stale or irrelevant, the agent retires it and builds a new one. If the agent needs to be rebuilt from scratch, the log is the source it reconstructs from. This is the event sourcing pattern applied to agent cognition.  

What you might be noticing now is that the agent itself needs infrastructure to support its function.  Much like a cell nucleus needs organelles to survive.  In this metaphor, the producer and consumer spawns are the appendages the agent "nucleus" uses to communicate with the outside world.  The log is its long-term memory, the derived stores are its working set.  All together they form an "agent cell", a term I've coined to describe a deployable grouping of an agent along with its collection of memory, storage, and producer/consumer adapters.  

One thing I want to acknowledge honestly: running an agent cell/event-log infrastructure at the scale implied by this architecture is expensive.  This is not the kind of architecture I'd propose at a three-person startup with a handful of dbt models.  Finding the point at which the ROI for doing things this way outweighs the cost of running a bunch of Kafka clusters is a genuinely interesting problem, one that makes the numerical analysis/systems design nerd in me overly excited. The answer today is certainly that this only makes economic sense at larger scales. But that boundary moves as compute gets cheaper, and it moves faster if agent cells can optimize their own infrastructure costs by pruning unused topics, retiring idle clusters, and right-sizing replication. The system that organizes its own topology can also optimize its own bill.

![Self-organizing agent cell mesh](/images/agent-mesh.png)

## The semantic governance layer is the top of the stack

It's hard enough to get humans to agree on ubiquitous language sometimes, one can only imagine how much worse this problem is when you introduce a bunch of autonomous, mostly unsupervised agent cells into the mix.  What does "active user" mean?  How do we define "revenue"?  What shape is a "customer" when your CRM, billing, and operational systems all see it differently?  

Consider what happens when an agent nucleus (LLM decision-maker) thinks "I want to count monthly active users" and it incorrectly conflates two conflicting definitions of that concept from two different subsystems when doing the join.  One system encodes "a monthly active user is someone who has logged in in the last month".  The revenue system encodes "a monthly active user is someone who has paid us money in the last month".  The agent cell responsible for reporting needs to navigate this nuance and use agreed-upon definitions specified at the organization level.  Otherwise the risk is that these systems produce numbers that are subtly wrong, or don't match up with human expectations.  

Academic research supports this. [Agent-OM](https://arxiv.org/abs/2312.00326), published at VLDB, applied LLM agents to ontology matching, the task of determining whether two schemas describe the same concept. The results were promising but not yet reliable without human oversight. Recent work on [LLM-supported collaborative ontology design](https://www.frontiersin.org/journals/big-data/articles/10.3389/fdata.2025.1676477/full), published in Frontiers, confirms that the problem is tractable but far from solved. 

And so the semantic layer responsible for disambiguating concepts and aligning agent understanding with human guidance is essential and sits at the top of the stack.  It is the highest-value component in the A2A data paradigm, and the hardest to automate away.  In fact it might never be, because it requires encoding human judgement about what concepts mean.  

What's truly interesting to me is that this happens to be the layer that provides a natural place to govern the operating costs for our self-organizing agent cell mesh.  Agent nuclei have no intuition about whether the streaming join they just provisioned creates a shuffle nightmare, or whether other cells need the exact same repartitioning in order to solve a different problem.  Each cell is locally correct, but naively duplicates work that could be pushed upstream to reduce the cost dramatically.    

Now multiply that by dozens of agent cells, each provisioning their own streaming topologies with their own join patterns, none of them aware of what the others are doing. Cell A reshuffles the sales stream by order_id. Cell B reshuffles the same sales stream by product_category. Cell C reshuffles it by billing_region. The same source data is being redistributed across the network three different ways simultaneously, by three agent cells that don't know each other exist.  

That's the cost governance problem. Each cell is locally rational, it built the topology it needed. But globally the system is doing redundant work that no single cell can see. This is where the semantic/governance layer needs to operate: not just governing what concepts mean, but governing how data physically moves. "These three cells all need the sales stream repartitioned.  Can we share a single repartitioned changelog instead of running three independent shuffles?"  Someone or something needs to quantify the relative cost for different architecture permutations.  The more autonomous the infrastructure provisioning becomes, the more critical automated cost governance becomes.

There's a huge amount of depth here that I want to explore in further posts.  For now, I will point out that the optimization the shared governance layer can't do is eliminate the fundamental work of repartitioning. Three different partition keys means three different physical arrangements of the data. But what it can do is ensure that each unique repartitioning happens exactly once, the source is read minimally, and high-demand repartitions are maintained proactively rather than created on demand.

## What we don't know

It's always fun to put on the speculation hat and dream about what comes next, and this post does a lot of that, so I want to be transparent about what we know today and what I'm guessing about.  

The governance requirement for durable event logs is real and driven by the operational reality of autonomous systems. If you're deploying agents that make consequential decisions, you need the log. That's not speculative.

The self-organizing agentic mesh / agent cell architecture is 100% speculative.  I've described a spectrum of "agents subscribe to and create topics dynamically" to "agents deploy their own infrastructure and curate their own knowledge bases".  The former is feasible today, the latter is further out.  Effort should be focused on the former.  That being said, grouping an agent with its supporting infrastructure as a deployable agent cell just makes logical sense.  It's simply another level of lego blocks. 

The semantic governance layer is probably years away according to current research.  "LLM agents can match ontologies with human oversight" is a long way from "agents autonomously agree on what revenue means across a network of 200 services." This is the hardest unsolved piece and the biggest risk to the entire thesis.

And cost governance is wide open.  I've described the problem and the shape of what a solution needs: a system that understands both what data means and what it costs to move.  That system doesn't exist yet.  

I plan to write more about the practical mechanics of agent-orchestrated consumers, the economics of self-organizing event infrastructure, and the convergence of semantic and cost governance. There's a lot to unpack.  But the foundation looks like this: the event log as the source of truth, agents as the compute and orchestration layer, the warehouse as the largest derived view, and semantic contracts as the governance layer that holds it all together. All the pieces are familiar, but the next evolution of the organism they assemble is not.

---

*Neil Turner is a Staff Data Engineer. He's spent 15 years building event-driven data platforms at Microsoft, Amazon, StockX, and May Mobility.*
