# IBM Instana Technical Review — Full Presenter Script
### 32 Slides · ~35–40 Minutes · Leadership Audience

> **How to use this document:** Each slide has a **core message** (what the slide must land), a **narration** (what to actually say — expand naturally, don't read verbatim), a **depth add-on** (extra technical substance that goes beyond the slide — this is what signals mastery, not just preparation), and a **transition** (the bridge line into the next slide, so you never go silent between slides).
>
> **Delivery philosophy:** Every slide's speaker notes in the deck itself give you the short version. This script gives you the *long* version — enough substance that if a senior lead interrupts with a follow-up mid-slide, you already have the answer ready.

---

## Before You Start

- [ ] Deck open in Presenter View — you want to see your notes, not just the slide
- [ ] Instana UI open in a background tab, logged in, ready for the live demo at the end
- [ ] Water nearby — this is a 35–40 minute delivery, pace yourself
- [ ] Know your audience: this is a **technical review**, not a sales pitch. Leaders in the room want to see you *understand* the trade-offs, not just that you can operate the tool.

**Opening line before Slide 1 (before you even advance):**
> "Thank you for the time. Over the next 35 minutes I want to walk through everything I built in this program — not just what worked, but the trade-offs, the failures I hit, and an honest assessment of where the platform's limits actually are. I'll close with a live demo of the environment running."

---

# PART 01 — FOUNDATIONS (Slides 1–8)

---

### Slide 1 — Title

**Core message:** Set the tone — technical rigor, not a sales pitch.

**Narration:**
> "This is IBM Instana Observability — a self-hosted deployment I built end to end, from a bare 3-node foundation up to a 5-node production cluster with three additional monitoring regions across Saudi Arabia. Nine delivery guides came out of this program, and I want to walk you through the reasoning behind every major decision — not just the happy path."

**Depth add-on:** If asked "why self-hosted and not SaaS" immediately — hold that answer for Slide 7, but you can preview: *"That's actually one of the first decisions I want to walk through — there's a real trade-off there, not an obvious choice."*

**Transition:** "Let me start with the agenda, so you know exactly where we're headed."

---

### Slide 2 — Agenda

**Core message:** Structure builds confidence before content does.

**Narration:**
> "Six parts. We'll start with foundations — what Instana actually is and how it works. Then the build itself, in the order I actually executed it. Third, synthetic monitoring — proactive testing, which is a meaningfully different capability from everything before it. Fourth is the part I want to spend the most care on: scaling the cluster from 3 to 5 nodes, and a genuinely honest assessment of high availability — what's resilient and what isn't. Fifth, two supporting tools that don't get talked about enough — Nexus and KubeSpray — because they're what made multi-region deployment repeatable instead of manual. And we'll close with a live demo of the environment running."

**Depth add-on:** Note explicitly that Part 4 (Scale & Reliability) is where you'll be most rigorous — this primes the audience that you're not going to oversell anything, which builds credibility before you even get there.

**Transition:** "Let's start at the foundation — what problem does observability actually solve?"

---

### Slide 3 — The Problem Observability Solves

**Core message:** Ground the technical content in a real operational scenario before diving into features.

**Narration:**
> "Fifty servers, ten services, one database cluster. Something breaks. The traditional approach is manual — SSH into boxes one at a time, grep through logs, guess at correlation under pressure. None of that scales, and none of it is fast. Instana's value proposition is a single connected view — metrics, traces, and logs auto-correlated, so the cause is usually one click away instead of an hour of investigation."

**Depth add-on:** *"This isn't a hypothetical — the incident response time difference between 'grep across 50 hosts' and 'one connected trace view' is measured in the difference between minutes and hours, and that gap compounds every time it happens."*

**Transition:** "So what specifically makes Instana able to do that? Four things."

---

### Slide 4 — What Is IBM Instana?

**Core message:** Define the product precisely — four specific, falsifiable claims, not marketing language.

**Narration:**
> "Four claims, and I want to be precise about each because vague claims are easy to make. First, zero configuration — install one agent, and every running technology on that host is auto-discovered. No manual instrumentation. Second, one-second granularity — most monitoring tools sample every 30 or 60 seconds; Instana captures every second, which means a five-second CPU spike that self-heals is actually visible instead of averaged away. Third, full-stack correlation — infrastructure, application performance monitoring, real user monitoring, and synthetic testing all sit in one data model, so you can click from a browser error straight down to the backend trace that caused it. Fourth, AI-assisted root cause analysis — automated anomaly detection across every monitored entity."

**Depth add-on:** *"The one-second granularity point is worth dwelling on — most competing tools use time-series databases optimized for storage efficiency at the cost of resolution. Instana's architecture makes a different trade-off: higher storage cost, but nothing gets averaged away before you see it."*

**Transition:** "That's the what. Let me show you the how — the actual pipeline underneath."

---

### Slide 5 — How Instana Works — The Pipeline

**Core message:** Demonstrate architectural literacy, not just operational familiarity.

**Narration:**
> "Five stages. Agents sit on every monitored host, auto-discovering and instrumenting. They send data to acceptors — separate ingestion endpoints for host agents, browser EUM beacons, synthetic test results, and OpenTelemetry data. From there it goes into Kafka, which is the piece people often don't expect — Kafka exists specifically to absorb bursts of traffic without back-pressuring the agents. Processors then consume from Kafka to build service maps and enrich the data, and finally it lands in five specialized datastores."

**Depth add-on:** *"The reason Kafka sits in the middle rather than writing directly to the datastores is backpressure isolation — if Elasticsearch or Cassandra is under load, Kafka buffers so agents never stall waiting on writes. That's a deliberate architectural choice, not an accident of implementation."*

**Transition:** "Which brings us to those five datastores — and why there are five, not one."

---

### Slide 6 — Five Purpose-Built Datastores

**Core message:** Show you understand *why* the architecture is polyglot, not just that it is.

**Narration:**
> "Elasticsearch holds traces, spans, events, and logs — it's built for full-text search across unstructured data. Cassandra holds metrics — pure time-series data optimized for extremely high-speed writes. ClickHouse handles analytics and aggregations using columnar storage, which makes rollup queries fast. Kafka, which we just covered, is the ingestion buffer. And PostgreSQL holds configuration, users, and alerts — relational data that needs consistent reads."

**Depth add-on:** *"This matters beyond theory — it's exactly why mixing these workloads on shared disks degrades all of them. Elasticsearch's write pattern and Cassandra's write pattern are fundamentally different I/O profiles, and that fact directly drove one of the architecture decisions in the 5-node migration, which I'll cover later."*

**Transition:** "Now — where do you actually run all of this? There are three deployment models."

---

### Slide 7 — Three Deployment Models

**Core message:** Show deliberate reasoning behind the Standard Edition choice — not "it's what we had."

**Narration:**
> "SaaS — IBM hosts everything. Fastest time to value, built-in high availability, but it requires your data to leave your perimeter and reach IBM's cloud. Standard Edition — self-hosted, using a tool called stanctl on top of k3s. Full control, data stays on-premises, scales from 1 to 5 nodes. This is what I deployed for this program. Custom Edition — self-hosted on OpenShift or Kubernetes directly, with full high availability through clustered datastores and pod-level failover — but it requires real Kubernetes operational expertise."

**Depth add-on:** *"The choice of Standard Edition over Custom Edition specifically was about matching operational maturity to the deployment — Custom Edition's HA guarantees are real, but they assume a team already comfortable operating Kubernetes at a deep level. Standard Edition is the right starting point, and I'll show exactly where its ceiling is later, rather than pretending it doesn't have one."*

**Transition:** "Standard Edition itself scales — let's look at exactly how."

---

### Slide 8 — Topology: 1, 3, and 5 Nodes

**Core message:** Set up the central thesis of the whole presentation — scaling nodes ≠ high availability. This sentence needs to land now so Part 4 has maximum impact later.

**Narration:**
> "Same software at every tier — one node for proof of concept, three for medium production, five for larger production. What changes is workload distribution: at five nodes, ClickHouse gets pulled onto its own dedicated node instead of sharing with the other datastores. One row here is worth pausing on — application-level high availability is 'No' at every single tier, including five nodes. That's deliberate on IBM's part, and it's the single most important technical finding I want to walk through today, later in the presentation."

**Depth add-on:** *"Notice the k3s control-plane-HA row is 'Yes' starting at 3 nodes — that's because stanctl deploys the underlying Kubernetes control plane in embedded etcd mode with three members, which gives you consensus-based resilience at the infrastructure layer. That's real, and I confirmed it directly against the running cluster. But it's a completely different layer from whether the Instana application itself survives a node failure — and conflating those two is the single most common misunderstanding I want to clear up today."*

**Transition:** "With the foundations covered, let's move to what was actually built — starting with the 3-node deployment itself."

---

# PART 02 — THE BUILD (Slides 9–14)

---

### Slide 9 — Divider: The Build

**Narration:**
> "From this point forward, everything I show you is something I personally deployed, broke, and fixed — not a vendor slide."

**Transition:** "Here's the full environment, in one view."

---

### Slide 10 — The Reference Environment, End to End

**Core message:** Establish the complete scope before drilling into individual phases.

**Narration:**
> "Five-node Instana backend on the left — gateway and UI on instana-0, datastores on instana-1, general workloads on instana-2, ClickHouse isolated on instana-3, additional agent workloads on instana-4. In the middle, everything being monitored — a Windows host running a purpose-built Node.js application, browser-based real user monitoring, and VMware vCenter. On the right, the synthetic and supply-chain layer — the original PoP plus three KubeSpray-provisioned PoPs across Saudi Arabia, and a Nexus repository mirror underneath everything."

**Depth add-on:** *"The thing that ties all of this together operationally is that it's still one Instana backend — one UI, one URL, one agent key across every single component, including the three new regional PoPs. That single-pane-of-glass property doesn't happen automatically — it required consistent DNS and firewall patterns across every new node type, which I'll touch on as we go."*

**Transition:** "Let's walk through the order this was actually built in."

---

### Slide 11 — The Build, In Order

**Core message:** Demonstrate methodical execution — this wasn't ad hoc.

**Narration:**
> "Eight phases, each producing its own standalone runbook. Three-node deployment first — VM creation, OS hardening, storage, networking, the stanctl install itself. Then the Windows agent and EUM. Then vCenter integration. Fourth, synthetic monitoring on the original PoP. Fifth, the 5-node migration. Sixth, a dedicated high-availability feasibility study — I treated that as its own deliverable because it deserved rigor, not a bullet point. Seventh, Nexus integration. Eighth, KubeSpray multi-region deployment."

**Transition:** "Let's go through the foundation phase in detail — the 3-node deployment."

---

### Slide 12 — Phase 1 — The 3-Node Foundation

**Core message:** Show operational discipline — the decisions that prevented downstream failures.

**Narration:**
> "RHEL 9 on VMware ESXi. Three nodes, each running as a k3s control-plane and etcd member from day one — that embedded HA mode I mentioned earlier. instana-0 handles the gateway, UI backend, and all acceptors. instana-1 holds Elasticsearch, Cassandra, Kafka, and PostgreSQL. instana-2 takes the remaining workloads."

**Then walk through the four key decisions, deliberately:**
> "Four decisions here shaped everything downstream. LVM partitioning with dedicated volumes, never automatic partitioning — that gives control over growth per workload type. A firewall internal-access zone configured on every node, every single time a new node joins — I'll come back to why that specific discipline mattered later. Clean OS installs only, never cloning — cloning a live node carries forward broken k3s state that causes subtle join failures. And critically, checking the stanctl version before any lifecycle operation — versions before 1.10.4 have a known deadlock condition after a 'stanctl down' command that leaves you with no easy path back."

**Depth add-on:** *"That stanctl version check sounds like a minor detail, but it's the kind of thing that turns a routine maintenance window into an extended outage if missed — a ten-second check that prevents a genuinely unrecoverable state."*

**Transition:** "With the foundation running, the next phase brought in a non-Linux workload — a Windows host with full application tracing."

---

### Slide 13 — Phase 2 — Windows APM & Browser Monitoring

**Core message:** Show cross-platform capability and a real debugging story — evidence of hands-on depth, not just following documentation.

**Narration:**
> "I built ShopLab specifically for this — a Node.js application with an in-browser EUM test lab that exercises every real-user-monitoring capability. Three things came out of this phase: full Windows infrastructure metrics from a non-Linux target, automatic distributed tracing with zero application code changes, and real user session monitoring — actual page loads, JavaScript errors, custom business events."

**Then the EUM port lesson — tell it as a story, not a bullet list:**
> "I want to walk through a specific lesson from this phase, because it's a good example of the kind of thing you only learn by doing it. Changing the EUM acceptor port — in this case from 443 to 446 — looks like a single configuration change. It isn't. It requires updating both sides simultaneously: the backend, via a stanctl command, and the frontend, in *two* separate places — both the reporting URL and the script source URL for the EUM loader itself. Miss either half, and monitoring fails completely silently. No error, no alert — data just stops arriving."

**Depth add-on:** *"That silent-failure characteristic is actually the important part of this story — it's not that the task is hard, it's that the failure mode gives you zero signal. That's exactly the kind of edge case that separates documentation-following from operational fluency."*

**Transition:** "The next phase brought VMware infrastructure into the same view."

---

### Slide 14 — Phase 3 — VMware vCenter Monitoring

**Core message:** Demonstrate security-conscious implementation, not just "it works."

**Narration:**
> "Instana reads live vSphere infrastructure directly through the vCenter API — no agents installed on ESXi itself. Four things had to be right. A read-only vCenter role, assigned at the vCenter root rather than per-object, which is what's actually required for full topology visibility. Certificate trust — vCenter's certificate imported into the Instana agent's own truststore, not the OS-level trust store. A single feature flag enabled through stanctl. And the result lives under a dedicated Platforms tab — deliberately separated from general Infrastructure, since vSphere entities have their own topology model."

**Depth add-on:** *"The read-only-at-root detail matters for a security review — it means Instana never has write access to vCenter, and the scope is broad enough to see the full inventory without needing per-object permission grants, which would be an operational burden to maintain."*

**Transition:** "That covers the build itself. Now let's move to a fundamentally different capability — proactive testing rather than passive observation."

---

# PART 03 — SYNTHETIC MONITORING (Slides 15–18)

---

### Slide 15 — Divider: Synthetic Monitoring

**Narration:**
> "Everything so far reacts to real traffic that's already happening. This next part is architecturally different — it finds problems before any real user ever encounters them."

**Transition:** "Here's the distinction in practice."

---

### Slide 16 — Passive vs. Proactive Monitoring

**Core message:** Establish the conceptual split clearly before getting into implementation.

**Narration:**
> "Passive monitoring — EUM and APM — watches real traffic as it happens. It's powerful, but by definition it requires an actual user to trigger a failure before you're aware of it. You're reactive. Proactive, synthetic monitoring runs scripted tests on a schedule from a dedicated location — HTTP checks, browser automation, SSL certificate validation, DNS resolution — completely independent of whether a real user is present. You find out before anyone calls you, rather than because someone called you."

**Transition:** "One location running synthetic tests is a start — but it's not enough on its own. Here's why."

---

### Slide 17 — Why More Than One Location?

**Core message:** This is a genuinely clever diagnostic capability — walk through the three scenarios carefully, this is a strong technical moment.

**Narration:**
> "Consider a single test location in Riyadh, reporting green — 200 milliseconds, everything looks fine. Meanwhile, users in Jeddah are reporting timeouts. A single-location test is structurally blind to this — there's no way to distinguish 'the application is down' from 'the network path to one specific region is broken.' Add a second location in Jeddah, and now you get diagnostic power: if Riyadh passes and Jeddah fails, that's very likely a regional network issue, not an application failure. If both fail, that confirms a genuine, global outage. Same signal, completely different response — and you only get that distinction with multiple locations."

**Depth add-on:** *"This is exactly the reasoning that justified deploying three separate regional PoPs later in the program — Riyadh, Jeddah, and Dammam — rather than treating one location as sufficient."*

**Transition:** "There's also a subtler architectural question — why does the test infrastructure itself need to be fully isolated from the backend it's testing?"

---

### Slide 18 — Why Test Locations Run Fully Isolated

**Core message:** Four independent, rigorous engineering reasons — this shows systems thinking, not just following a deployment guide.

**Narration:**
> "Four reasons, and any one of them alone would justify the isolation. First — honest network perspective: if the test runs from inside the backend's own network, it bypasses the load balancer and firewall that real users go through, and the result stops reflecting reality. Second — no shared failure domain: if the PoP and the backend share infrastructure, they fail together, which means you lose exactly the alert that was supposed to catch the backend failure in the first place. Third — resource isolation: browser-based synthetic tests are genuinely compute-intensive, and isolating them protects the datastores that production applications depend on. Fourth — independent survivability: a PoP's entire job is to detect and report backend outages, which is structurally impossible if it depends on that same backend to function."

**Depth add-on:** *"The second point — shared failure domain — is the one people underestimate most. It's a subtle failure mode: everything looks configured correctly, tests pass in normal operation, and the gap only reveals itself during the exact incident you built the system to catch."*

**Transition:** "That covers synthetic monitoring conceptually. Now — the part of this program I want to be most rigorous about: scaling the cluster, and an honest reliability assessment."

---

# PART 04 — SCALE & RELIABILITY (Slides 19–23)

---

### Slide 19 — Divider: Scale & Reliability

**Narration:**
> "I want to be precise here rather than optimistic. This is the part of the program where the findings genuinely matter for any production decision."

**Transition:** "Let's start with the migration itself."

---

### Slide 20 — Scaling From 3 to 5 Nodes

**Core message:** Show operational precision on a genuinely high-risk procedure.

**Narration:**
> "This is an IBM-documented, officially supported migration path — not something improvised. What actually changes: the 1200 gigabyte analytics disk physically moves from instana-1 to a new instana-3, which then becomes a dedicated node for ClickHouse — 8 cores, 32 gigabytes, entirely its own. Critically, this requires zero agent reconfiguration — domain, UI URL, and every agent key stay exactly as they were."

**Then slow down for the risk:**
> "The highest-risk step in the entire migration is the ESXi disk operation itself. There are two options when detaching that analytics disk in vSphere, and they look almost identical in the UI: 'Remove from VM' is correct — it detaches the disk while preserving the data. 'Delete files from datastore' is the wrong option, and it is permanent, unrecoverable data loss. One click, wrong option, and eighteen months of analytics history is gone. I also want to flag — stanctl has to be at version 1.10.4 or later before this migration starts; earlier versions have a known deadlock condition after the 'down' command in this specific workflow."

**Depth add-on:** *"I treat this disk-move step the way you'd treat any operation with an irreversible failure mode — documented runbook, version pre-check, and a maintenance window sized generously enough that nobody feels rushed through the one step that can't be undone."*

**Transition:** "That's the mechanics of scaling. Now the more important question — does scaling actually solve the problem most people are really asking about?"

---

### Slide 21 — Scalability Is Not High Availability

> **This is the analytical centerpiece of the whole presentation. Slow your pace here. Make eye contact. This is the slide the audience should remember afterward.**

**Core message:** The single most important finding of the entire program.

**Narration:**
> "When someone asks 'should we add more servers,' what they're very often actually asking is 'what happens when one of our servers dies.' Those are two different questions, and I want to answer both precisely rather than let one stand in for the other. Scalability adds capacity — it spreads workloads across dedicated hardware, and it solves the problem of a system being slow under load. High availability adds resilience — it runs duplicates so that a single failure has zero impact, and it solves a completely different problem: the system must never go down at all."

**Then the finding itself — say this slowly:**
> "Here's the finding: a five-node Instana deployment can still go down completely — exactly the same as a three-node deployment — if the wrong single node fails. Scaling from three nodes to five nodes did not change that fact. It added capacity. It did not add resilience at the application layer. And I want to be direct about that distinction, because I think it's the most consequential finding to come out of this entire program."

**Depth add-on:** *"Standard Edition does provide real resilience — but only at the k3s control-plane layer, not for the Instana application itself. That's not a flaw in the product; it's a deliberate architectural trade-off. The next slide shows exactly where that line sits, verified against our own running cluster rather than asserted from documentation."*

**Transition:** "Let's look at exactly what is and isn't resilient — with direct evidence from the cluster."

---

### Slide 22 — What's Resilient — Verified Against the Live Cluster

**Core message:** Evidence-based, not theoretical — this is where you demonstrate you didn't just read documentation, you tested the claim.

**Narration:**
> "On the resilient side: the k3s control plane itself. instana-0, instana-1, and instana-2 all run as etcd members — that's a three-node consensus group, which tolerates one node failure while maintaining quorum. I confirmed this directly — running `kubectl get nodes` shows all three explicitly labeled control-plane, etcd, master. If any single one of those three goes down, cluster scheduling and `kubectl` operations continue without interruption. That's real, and it's verified, not assumed."

**Then the other side — deliver this with the same evidentiary rigor, no hedging:**
> "On the other side: everything the Instana application itself runs — the UI, the gateway, the acceptors, and all five datastores — exists as a single instance pinned to exactly one node. There is no failover at this layer. If instana-1 goes down, Elasticsearch, Cassandra, Kafka, and PostgreSQL all go down with it, and there's no redundant copy anywhere in the cluster to take over."

**Explain the root cause — this is where deep understanding shows:**
> "The root cause is specific: stanctl uses what's called a local-path storage provisioner, which ties every data volume to the physical disk of one specific node. That's a deliberate performance trade-off, not a k3s limitation — local disk access is faster than networked storage, and IBM chose performance over redundancy for Standard Edition's default configuration."

**Transition:** "Given that finding, the natural next question is: what are the actual paths to close that gap? There are three, realistically."

---

### Slide 23 — Three Realistic Paths to True HA

**Core message:** Turn the honest limitation into a constructive, ranked set of options — shows solution-oriented thinking, not just problem identification.

**Narration:**
> "Ranked by what each option actually requires and delivers. First, and the recommended starting point: vSphere HA at the hypervisor level. This requires no changes to Instana at all — if a VM fails, vSphere restarts it automatically on another host. Recovery is around five minutes, it needs shared storage, and it costs nothing extra in Instana licensing. Second: a share-nothing active-active configuration — two fully independent backend deployments, with every agent reporting to both simultaneously. This one comes with a caveat — it's only documented through a 2022 community forum post, not official IBM documentation, and it means no unified view and manual configuration sync between the two backends. Third: Custom Edition on OpenShift — genuinely a different product, with clustered datastores and automatic pod rescheduling on node failure. That's the path to true zero-downtime, but it requires an OpenShift license, a different Instana SKU, and real Kubernetes operational expertise."

**Depth add-on:** *"I'd recommend vSphere HA as the practical next step for this environment specifically — it's the lowest-effort option that closes the most common failure scenario, a single VM or host going down, without requiring any architectural changes to what's already deployed."*

**Transition:** "That completes the reliability assessment. Now I want to cover two supporting technologies that don't get enough attention but were essential to making this program's later phases repeatable — Nexus and KubeSpray."

---

# PART 05 — SUPPORTING TOOLS & TECHNOLOGIES (Slides 24–28)

---

### Slide 24 — Divider: Supporting Tools & Technologies

**Narration:**
> "Neither of these is an Instana-specific tool — I want to define each one precisely before showing exactly how it was used, because it's easy to conflate them with the Instana-specific components we've already covered."

**Transition:** "Starting with Nexus."

---

### Slide 25 — What Is Nexus Repository?

**Core message:** Precise, standalone definition — treat this the way you'd explain any general-purpose infrastructure tool to a technical audience unfamiliar with it.

**Narration:**
> "Sonatype Nexus is a general-purpose artifact repository manager — nothing about it is specific to Instana. In plain terms, it's a self-hosted server that sits between your infrastructure and the public internet, storing, proxying, and serving software packages and container images. There are three repository types worth knowing. A proxy repository caches artifacts from a remote source — Maven Central, Docker Hub — on the first request, and serves the cached copy on every request after that. A hosted repository stores artifacts you push directly yourself. And a group repository combines multiple repositories behind a single URL, so clients only need to know one endpoint regardless of where the actual artifact lives."

**Depth add-on:** *"The proxy-repository caching behavior is the mechanism that matters most here — the first host to request an artifact pulls it from the internet and populates the cache; every subsequent request, from any host, is served locally at LAN speed instead of going back out to the internet."*

**Transition:** "Here's specifically what we used it for in this deployment."

---

### Slide 26 — Nexus in This Deployment

**Core message:** Show that Nexus wasn't caching for its own sake — three distinct, justified use cases.

**Narration:**
> "Three integrations. First, a Maven proxy specifically for agent sensor and feature bundle updates — this is an IBM-documented pattern for this exact use case. Second, a Docker registry spanning seven separate ports — covering icr.io, k8s.io, docker.io, quay.io, and the container images used by the synthetic PoPs. This one specifically supports air-gapped operation, which matters for any environment with restricted internet access. Third, a YUM and RPM proxy for the Linux packages themselves — popctl, stanctl, and the instana-agent packages. That third one is an independent Nexus capability, not something IBM documents specifically, but it completes the picture — every artifact type this environment needs now flows through one controlled internal source instead of hitting the public internet directly from every node."

**Transition:** "The second supporting technology solved a different problem entirely — provisioning three independent Kubernetes clusters, repeatably."

---

### Slide 27 — What Is KubeSpray?

**Core message:** Another precise, standalone definition — and critically, explain why it's *not* the same as k3s or popctl, since that's the natural point of confusion.

**Narration:**
> "KubeSpray is an open-source, Ansible-based tool for deploying and managing production-grade Kubernetes clusters — it's a CNCF-associated project, vendor-neutral, and it provisions real, standards-compliant Kubernetes clusters using kubeadm underneath. That's an important distinction from k3s, which is a lightweight, opinionated Kubernetes distribution. Four things define it: it's inventory-driven, meaning cluster topology — hosts, roles, network plugin choice — is defined declaratively in version-controlled YAML. It's kubeadm-based, so the resulting clusters are standard, not a stripped-down variant. It's repeatable by design — running the same playbook against a new inventory produces an identical, fully independent cluster. And specifically why I chose it here: this program needed three fully independent, single-purpose clusters, one per city, not one shared cluster split across regions."

**Depth add-on:** *"The declarative, inventory-driven property is really the whole value proposition — the cluster definition lives in version control, not in someone's memory of the commands they ran, which is exactly what makes standing up a fourth or fifth region later a matter of adding an inventory file, not repeating a manual process."*

**Transition:** "Here's the concrete result — three independent clusters, live."

---

### Slide 28 — KubeSpray in This Deployment

**Core message:** Tie the abstract tool description to the specific, verifiable outcome — three online regions.

**Narration:**
> "Three cities — Riyadh, Jeddah, and Dammam — each running a fully independent, single-node Kubernetes cluster, provisioned from a single Ansible control point. All three are currently online. The choice of KubeSpray over popctl here was specific: popctl is built around provisioning a single PoP; standing up three fully independent clusters from one control point, repeatably, needed Ansible's declarative approach instead."

**Depth add-on:** *"Each of these clusters is fully independent — no shared control plane, no shared failure domain between regions — which directly reflects the isolation principles I covered earlier in the synthetic monitoring section. The architecture reasoning and the tooling choice are consistent with each other, not two separate decisions."*

**Transition:** "That completes the technical build. Let me close with a quick summary of scope, the lessons that will outlast this specific lab, and then move into the live demo."

---

# PART 06 — CLOSE (Slides 29–32) + LIVE DEMO

---

### Slide 29 — The Program, Quantified

**Core message:** A confident, factual summary — let the numbers speak, don't oversell them.

**Narration:**
> "Nine delivery guides. Eight production-configured nodes. Three live regions across the Kingdom. Seven security-hardened registry mirrors through Nexus. Five independently scalable node roles. And one evidence-based high-availability assessment, rather than a marketing claim about reliability."

**Transition:** "Before we move to the live environment, I want to close the technical review with the lessons that matter beyond this specific lab."

---

### Slide 30 — Lessons That Outlast This Lab

**Core message:** This is where you demonstrate genuine reflective engineering judgment — not just "what I did" but "what I'd tell the next person."

**Narration:**
> "Six lessons I'd want the next engineer touching this environment to know immediately. Firewall configuration first, always — the majority of 'mysterious' node-join and TLS failures in this program traced back to exactly one missing internal-access rule. Pods never read /etc/hosts — both CoreDNS and KubeSpray's nodelocaldns bypass it entirely for external domain resolution; dnsmasq turned out to be the durable pattern across both k3s and KubeSpray deployments. Version checks are non-negotiable — the stanctl 1.10.4 requirement isn't a formality, it prevents a genuine deadlock. Scaling is not resilience — the most consequential architectural distinction this program surfaced, and one I verified with direct evidence rather than assumed. Configuration changes are almost always two-sided — the EUM port change, image registry configuration, DNS — every case that looked like a single change actually had two places that needed updating together. And clean installs beat cloning, every time — every attempt in this program to clone a live node for a new role carried forward broken state that caused subtle failures later."

**Transition:** "With that, let's move from slides to the actual environment."

---

### Slide 31 — Live Demo Transition

**Narration:**
> "Everything I've described is running right now. Let me show you — infrastructure, application traces, real user sessions, VMware visibility, and synthetic tests across all three Saudi regions."

**Transition:** Switch to the browser / Instana UI now.

---

## LIVE DEMO SEGMENT (~10 minutes)

Follow this exact order — each stop should take 60–90 seconds:

1. **Infrastructure tab** — show the 5-node cluster, point out live metrics updating
2. **Services → a traced request** — click into a trace waterfall, narrate what each span represents
3. **Websites & Mobile Apps (EUM)** — show a real session, point out page load and any JS errors captured
4. **Platforms tab** — expand the vCenter tree, show a host and a VM
5. **Synthetic Monitoring → Locations** — show all locations Online, including Riyadh, Jeddah, Dammam
6. **A synthetic test result** — show pass/fail per location, tie back to the "why multiple locations" argument from Slide 17

**Narrate while navigating — never click silently.** Example: *"I'm going to open a trace now — watch how the waterfall breaks down exactly where the time was spent."*

**If something is slow to load:** *"While that loads — this is actually a good moment to mention..."* and fill with a relevant fact rather than going silent.

---

### Slide 32 — Thank You

**Narration:**
> "That's the full environment — architecture, the build itself, an honest reliability assessment, and the supporting tooling that made multi-region deployment repeatable. Nine documented guides, an evidence-based HA assessment, and three live regions. I'm glad to go deeper on any part of this — happy to take questions."

**Then stop talking.** Let the room lead into Q&A.

---

## Anticipated Questions — Have These Ready

**"Why didn't you go straight to Custom Edition if HA matters?"**
> "Standard Edition matches the operational maturity this program was scoped for — it gets you k3s control-plane resilience out of the box with a much lower operational burden. Custom Edition is the right answer once there's a genuine zero-downtime SLA requirement and a team ready to operate Kubernetes at that depth. I'd treat that as a deliberate next-phase decision, not a default."

**"How confident are you in the HA findings — is that just documentation, or did you test it?"**
> "Both. The etcd control-plane resilience claim is verified directly — `kubectl get nodes` on the running cluster shows all three original nodes labeled control-plane, etcd, master. The application-layer finding comes from understanding stanctl's storage provisioner architecture — it's a structural fact about how the local-path provisioner binds volumes to nodes, not something that needs a live failure test to confirm."

**"What would you do differently if you started over?"**
> "Two things. I'd set up the Nexus mirror and the DNS/dnsmasq pattern before the first node, not partway through — both ended up needed repeatedly across later phases, and doing them first would have saved rework. And I'd run the stanctl version check as the very first command of the entire program, not just before the migration — it's cheap insurance."

**"Is KubeSpray production-grade, or was this a shortcut?"**
> "It's a CNCF-associated, widely adopted tool — used in production by teams who need vendor-neutral, standards-compliant Kubernetes without committing to a specific managed offering. It was the right tool specifically because it's declarative and repeatable, which is exactly what standing up three independent regional clusters required."

**"What's the actual cost/effort of adding a fourth region?"**
> "With the KubeSpray pattern already established, it's an inventory file and a playbook run — the heavy lifting was building the repeatable pattern once, not each individual region."

---

## Delivery Notes

- **Pace:** This script is dense — you do not need to say every word. Use it as a depth reserve, not a transcript to recite.
- **The one slide to never rush:** Slide 21 (Scalability ≠ HA). Everything else can move briskly; that slide is the one the audience should remember.
- **Confidence signal:** When you don't know something in Q&A, say so precisely — *"That's outside what I directly verified — let me confirm and follow up"* — rather than guessing. That's more credible to a technical audience than a confident wrong answer.
