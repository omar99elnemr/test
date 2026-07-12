# IBM Cognos Analytics - Simplified Presenter Script

PRESENTER SCRIPT

IBM Cognos Analytics

Platform Overview, Self-Hosted Deployment & Capability Demo

Companion script to the 36-slide presentation deck

Full delivery time: approximately 35–45 minutes, plus Q&A

How to use this script

Each slide has three parts. Core message is the one idea the slide must land — keep it in view so you never lose the thread mid-sentence. Narration is natural spoken language: talk like you're explaining this to a colleague, not reading the slide aloud — the slide shows the words, your job is to say what they mean. Depth add-on is optional material to reach for if there's time, a follow-up question invites it, or the room's energy calls for more — skip it freely when running short.

Timing notes are guides, not a stopwatch — adjust to the room.

SECTION OPENING

Title & Agenda

Slides 1–2

SLIDE 1   Title — IBM Cognos Analytics

**Explain**

Good [morning/afternoon] everyone. Today I'm going to walk you through IBM Cognos Analytics — not as a marketing pitch, but as something we actually built, broke, fixed, and are now demonstrating live. We'll start with what the platform actually is and what it takes to run it, then I'll walk you through our real deployment — including the parts that didn't go smoothly — and finish with a live demo on real data.

The reason I'm structuring it this way is deliberate: by the time we get to the demo, you'll understand exactly what you're looking at and why it matters — not just that it looks nice.

**Extra (Optional)**

If asked why this matters as an IBM partner: this dual structure — technical rigor plus honest engineering — mirrors exactly how a real client engagement should be pitched. Clients increasingly see through polished-only demos; showing the real friction and how it was resolved builds more trust than a flawless-looking walkthrough ever could.

⏱ ~1 minute

SLIDE 2   Agenda

**Explain**

Here's the shape of the next 40 minutes. We'll start with the fundamentals — what Cognos actually is, the terminology you'll hear throughout, and where it sits in IBM's broader analytics portfolio. Then we'll cover how it's offered and what it takes to run, technically. From there we get into why this matters to a business — the actual case for change. Then the heart of the session: our real deployment journey, warts and all. And we'll close with a live demo on the platform we actually built.

**Extra (Optional)**

If the audience is more technical, you can flag now that Section 5 (the deployment journey) is the longest and most detailed — six real issues, each with root cause and resolution, several with actual screenshots from the build. If the audience is more business/client-facing, flag that Section 4 (the business case) and Section 6 (the demo) are the sections most relevant to them.

⏱ ~1 minute

SECTION 01

What is IBM Cognos Analytics

Slides 3–6

SLIDE 3   Section divider — What is IBM Cognos Analytics

**Explain**

Let's start from first principles. I'm not going to assume everyone in the room has worked with Cognos before, so this section covers what it is, the language we'll use throughout, and who it's actually built for.

**Extra (Optional)**

Nothing to add here — keep this brief, it's a two-second transition slide. Move quickly to slide 4.

⏱ ~15 seconds

SLIDE 4   Product Definition

**Explain**

At its simplest, Cognos Analytics is IBM's self-service business intelligence platform. The word that matters most here is 'governed' — anyone can build a chart in a dozen different tools, but Cognos's actual value proposition is that everyone in an organization works from the same trusted, permissioned version of the data. That's the difference between self-service and chaos.

It does three things in one environment: data modeling — that's building a reusable, governed layer on top of raw data; reporting and dashboards — the visual output people actually consume; and increasingly, AI-assisted insight — natural language querying and automated discovery.

And on the right, you can see it's not a standalone product — it's one piece of IBM's broader Business Analytics continuum, which also covers extended planning and financial close. Cognos is specifically the business intelligence layer of that story.

**Extra (Optional)**

If asked how this differs from Power BI or Tableau: the honest answer is that visually, modern Cognos dashboards are competitive with either. The differentiation IBM leans on is governance at scale and the semantic layer — Power BI and Tableau both have governance features, but Cognos's model was built enterprise-governance-first, whereas those tools grew governance features on top of a self-service-first product. That's a real, defensible distinction, not just marketing language — we'll actually demonstrate governance live in the demo later.

⏱ ~2 minutes

SLIDE 5   Terminology

**Explain**

Quick vocabulary check before we go further, because I'll use these terms constantly for the rest of the session and I don't want anyone lost. Content Manager is the brain of the system — it manages security, metadata, scheduling. The content store is the actual database behind Content Manager — in our case, PostgreSQL. A data module is the governed semantic layer we build on top of raw data — think of it as the translation layer between messy source data and clean, trustworthy reports. The dispatcher routes requests around the system. A gateway is an optional web-server front end — we didn't need one, and I'll explain why later. And a namespace is simply where your login credentials come from — could be LDAP, Active Directory, or Cognos's own built-in system.

**Extra (Optional)**

Worth flagging now, briefly, that 'namespace' becomes a major plot point later in this presentation — our biggest deployment challenge lived entirely inside this concept. You can plant that seed here ("you'll hear a lot more about namespaces in Section 5") to build anticipation without giving away the ending.

⏱ ~2 minutes

SLIDE 6   Audience & Fit

**Explain**

One more piece of context before we get technical: who is this actually for? The honest answer is almost everyone. Data and IT leadership care about governance and infrastructure. Sales and marketing leaders want visibility into pipeline and campaign performance. Operations wants supply chain and process visibility. The C-suite wants the aggregate picture. HR has its own reporting needs. And then there's a long tail of analysts and data scientists who use it as a working tool day to day.

The reason this matters for how we think about a deployment like ours: whoever ends up as the buyer, the actual usage is genuinely cross-functional — which is part of why governance matters so much. You're not securing one team's data, you're securing everyone's view into the same data.

**Extra (Optional)**

If presenting to a specific client with known priorities, you can tailor this live — point at whichever buying group in the room maps to your actual audience and spend an extra sentence on their specific pain point. This slide is designed to let you do that without derailing the flow.

⏱ ~1.5 minutes

SECTION 02

Offerings & Editions

Slides 7–9

SLIDE 7   Section divider — Offerings & Editions

**Explain**

Before we get into requirements and architecture, it's worth covering how Cognos is actually offered — because the deployment model we chose for this lab was a deliberate decision among real alternatives, not the only option.

**Extra (Optional)**

None needed — transition slide, keep it short.

⏱ ~15 seconds

SLIDE 8   Deployment Flexibility

**Explain**

Cognos runs four ways. SaaS is the fastest path — IBM manages everything, you just log in. On-premises — which is what we built — means you own the infrastructure and, critically, you control data residency and network posture completely. Certified Containers is the Kubernetes-native option, great for teams already running CI/CD pipelines. And Cloud Pak for Data integrates Cognos into IBM's broader data and AI platform.

We picked on-premises specifically because our brief was a restricted-network deployment — that requirement essentially rules out SaaS by definition, and on-premises gave us full control to design the network posture exactly the way we wanted.

**Extra (Optional)**

If asked whether Containers would have been a better choice than plain on-premises VM: it's a legitimate alternative, and worth acknowledging honestly — Containers would have given us faster redeployment and better resource isolation. We chose VM-based on-premises specifically to demonstrate the traditional deployment path most enterprise IT teams are still more familiar with, and because it let us show real OS-level troubleshooting — disk space, permissions, network config — that's directly transferable to how most client environments still actually look today.

⏱ ~2 minutes

SLIDE 9   Editions

**Explain**

Two editions. Standard covers the full core platform — reporting, dashboards, data modules, governance. Premium adds expanded AI capabilities, including Reporting Agents in supported environments — that's IBM's current big push, sometimes referred to as the agentic era of Cognos.

For a proof-of-concept like ours, Standard already demonstrates the vast majority of what a client needs to see. Premium's AI features are the natural next step in a follow-up conversation once the fundamentals are proven.

**Extra (Optional)**

If your build did select the Agentic AI install component and you plan to demo it: mention here that this specific lab actually included the Agentic AI component during installation, so you'll have something concrete to show later rather than just describing Premium in the abstract. If you didn't get to build that out, don't oversell it — just present this as informational.

⏱ ~1.5 minutes

SECTION 03

Architecture & Requirements

Slides 10–13

SLIDE 10   Section divider — Architecture & Requirements

**Explain**

This next section is where we get specific — the actual components involved, and the real hardware numbers behind them. This is the technical foundation everything in Section 5 builds on.

**Extra (Optional)**

None needed.

⏱ ~15 seconds

SLIDE 11   Architecture

**Explain**

Cognos breaks down into three tiers. The content tier is Content Manager plus its database — the system of record. The application tier is where the actual work happens — reporting, dashboards, data modules, queries. And the gateway is optional — a web-server front end you only need for single sign-on or when you're load-balancing across multiple application-tier nodes.

We ran a single-server deployment — all three tiers, well, two of the three since we skipped the gateway, on one machine. IBM's own documentation explicitly describes this as the standard approach for demonstration and proof-of-concept work, which is exactly what this was.

**Extra (Optional)**

If asked what a distributed deployment would actually look like: picture Content Manager on one dedicated node for resilience, then multiple application-tier nodes behind a load-balancing gateway for horizontal scaling under heavy report-rendering load. That's the natural evolution path from what we built — nothing about our single-server build would need to be redesigned from scratch, it would just be split across more machines.

⏱ ~2 minutes

SLIDE 12   System Requirements

**Explain**

Here's what it actually takes to run this. IBM's documented minimum is 4 CPU cores, 32 gigs of RAM, 18 gigs for the install plus 8 gigs of temp space. That's a real, working minimum — but it leaves you with essentially no headroom.

In practice, we'd recommend thinking in three tiers, which is what this table shows: the bare IBM minimum if you're doing a five-minute smoke test; a safe minimum with some real buffer for actual day-to-day use; and a recommended tier — 100 gigs or more of disk, 8-plus cores — if this is something you're going to use and iterate on over days or weeks, which is exactly the situation we were in.

One detail worth calling out specifically: the temp directory matters more than people expect. Every report render, regardless of output format, goes through temp space first. Run low on that, and you get failures that look completely unrelated to disk space unless you know to check it.

**Extra (Optional)**

This is a good moment to foreshadow Issue 1 in the deployment journey — you can say something like, “and as it turns out, disk space was actually the very first thing that stopped us cold — we'll get to that story in a few minutes.” That's a natural narrative hook that makes Section 5 land better when you get there.

⏱ ~2.5 minutes

SLIDE 13   Content Store Options

**Explain**

Cognos supports five databases as its content store: Db2, Oracle, SQL Server, Informix, and PostgreSQL. The first three are all genuine enterprise options with real setup overhead — DDL generation, specific encoding requirements, DBA involvement. Informix is actually only available bundled with the Windows-only Easy Install path, which immediately ruled it out for us since we were building on Linux.

We went with PostgreSQL. No licensing cost to track, open-source, and it matched the same self-managed philosophy we'd already used successfully elsewhere in this lab environment.

**Extra (Optional)**

If asked whether PostgreSQL caused any of our later problems: this is worth answering honestly and precisely when we get to Section 5 — PostgreSQL itself performed exactly as expected throughout; every issue we hit was either an OS-level configuration gap or a Cognos-side namespace-seeding issue, not a database compatibility problem. Worth having that distinction ready since it's an easy assumption to make incorrectly.

⏱ ~2 minutes

SECTION 04

Why It Matters

Slides 14–18

SLIDE 14   Section divider — Why It Matters

**Explain**

We've covered what Cognos is and what it takes to run technically. Now let's step back and talk about why any of this matters to a business — because a platform is only as valuable as the problem it solves.

**Extra (Optional)**

None needed.

⏱ ~15 seconds

SLIDE 15   The Business Case — Stats

**Explain**

Three numbers worth sitting with. Sixty-eight percent of enterprise data is never analyzed at all — that's not a small inefficiency, that's the majority of what organizations collect going completely unused. Fifteen percent — that's the actual employee adoption rate of BI software industry-wide, according to BARC's research. And eighty percent of executives don't trust their own data enough to act on it confidently.

Put those three together and you get the real business case for a platform like this: it's not about generating more charts. It's about closing the gap between data that exists, data that gets used, and data that gets trusted.

**Extra (Optional)**

If pressed on sourcing: Ventana Research's Analytics and Data Value Index (2021) is the source for the 68% figure, and BARC's Score Integrated Planning and Analytics Report (2023) for the 15% figure. The 80% distrust figure comes from an industry survey cited directly in IBM's own Cognos Analytics conversation guide — worth being upfront that this last one is IBM-sourced rather than independent, if asked directly, since intellectual honesty on sourcing matters more than the stat itself.

⏱ ~2 minutes

SLIDE 16   Why Change

**Explain**

This is a framework IBM uses to structure the conversation about why an organization should reconsider its current approach, and it has three parts. External pressures: data volume keeps growing, but a lot of organizations are still governing that data with not much more than spreadsheets. Current approach breaking down: analytics used to be one centralized team's job; today, every line of business expects to own its own insight, and they expect modern, AI-assisted tools to help them do it. And risk of doing nothing: poor data quality and low trust mean organizations sit on data they can't actually act on — even while drowning in more of it every quarter.

**Extra (Optional)**

This three-part structure — external pressures, current approach breaking down, risk of inaction — is a deliberate sales-conversation framework from IBM's own partner enablement material, not something we invented for this deck. If presenting to a genuine prospective client, this is the section where you'd pause and ask a version of “does any of this sound familiar in your organization?” — it's designed as a conversational hook, not just information to recite.

⏱ ~2.5 minutes

SLIDE 17   Why You

**Explain**

So if that's the problem, here's Cognos's specific answer, in three parts. AI-powered insights — built-in AI helps blend and clean data and surface trends, so people get answers in real time instead of waiting in a queue behind a data team. Integrated analytics — one environment for joining data, exploring it, and delivering the output, rather than five disconnected tools stitched together. And governance at scale — the same rules that work for one user work identically for ten thousand, so people only ever see what they're supposed to see.

**Extra (Optional)**

The governance point is worth emphasizing more than it might seem on the slide — it's actually the hardest of the three to fake or shortcut, and it's exactly what we demonstrate live later with the two-user permission test. If you want a stronger transition into the demo section, you can plant that here: “governance at scale is the one of these three we're actually going to prove live today, not just describe.”

⏱ ~2 minutes

SLIDE 18   Proof in Practice

**Explain**

Two real examples. In healthcare, one organization replaced around a hundred static reports across more than twenty care procedures with interactive dashboards — and during COVID, they stood up near-real-time dashboards in just one week. That responsiveness translated into three million dollars in additional annual government funding tied to improved emergency department KPIs.

In global finance, L'Oréal moved from a decentralized culture of manual data crunching to one harmonized platform — twenty-five hundred-plus users worldwide now working from a single, live view of the business, for both planning and reporting.

**Extra (Optional)**

If asked for the source of these case studies: both come from IBM's own official customer references and partner-enablement material — the L'Oréal quote is attributed to Laurent Mesnier, their FP&A Information Systems Manager. Worth being transparent that these are IBM-selected success stories rather than independently audited case studies, if a skeptical audience pushes on it — that transparency is more credible than overstating their independence.

⏱ ~2.5 minutes

SECTION 05

Our Deployment Journey

Slides 19–30 — the heart of the presentation

SLIDE 19   Section divider — Our Deployment Journey

**Explain**

This is the core of what I want to share today. Everything up to this point has been platform education — now we get into what actually happened when we built this. I want to be upfront: we hit four distinct issues along the way, and I'm going to walk through every one of them — what broke, how we figured out why, and how we fixed it.

I'm doing this deliberately, not because I have to. A clean, friction-free demo is easy to fake and doesn't tell you much about how this platform behaves under real conditions. What I'm about to show you is more useful than that.

**Extra (Optional)**

This is a good moment to set audience expectations on tone: signal explicitly that showing real friction isn't a weakness in the presentation — it's the most credible part of it. If the audience is client-facing/sales-oriented, this framing is what turns “we hit problems” into “we do real, rigorous engineering, and here's the evidence.”

⏱ ~1.5 minutes — don't rush this transition, it sets up the whole section's credibility

SLIDE 20   Deployment Decisions — Platform, Topology, Network

**Explain**

Before we get into what broke, I want to show the decisions we made deliberately, and why — because a lot of what looks like an arbitrary technical choice was actually a reasoned trade-off.

We picked AlmaLinux 9 over Windows because it matched our existing lab environment, and because Windows only unlocks Easy Install, which IBM's own documentation flags as not recommended for production. We picked single-server over a distributed topology because IBM explicitly documents single-server as the standard approach for demonstration and proof-of-concept work — which is exactly what this was; a distributed setup would have solved problems we didn't actually have, like Content Manager failover or a report-volume bottleneck. And we deliberately chose no direct internet access, routing everything through an internal Nexus repository — that's a real, defensible security posture, not just ‘we didn't have Wi-Fi.’

**Extra (Optional)**

The Nexus-proxied network decision is worth spending an extra beat on if the audience is security-conscious: it directly demonstrates least-privilege egress and package provenance, which are real enterprise security requirements, not just an academic lab restriction. This is the version of “restricted network” that's actually explainable and sellable to a real client — versus something that would just look like an arbitrary obstacle.

⏱ ~2.5 minutes

SLIDE 21   Deployment Decisions — Database, Gateway, Authentication

**Explain**

Three more decisions. PostgreSQL for the content store — no licensing to track, consistent with the open-source approach we'd already used successfully elsewhere. No dedicated gateway, because the application tier already handles that role unless you specifically need single sign-on, which was out of scope here.

And authentication — this one's worth pausing on, because it's not a clean, one-line decision. We didn't have any existing LDAP or Active Directory service in the lab, so we started with Cognos's own built-in namespace, since that's the simplest supported path with no external infrastructure needed. It turned out not to work, for reasons we genuinely couldn't fully explain even after ruling out a long list of possible causes — I'll walk through exactly what we checked in a few minutes, and what we did about it.

**Extra (Optional)**

Deliberately don't fully resolve the authentication thread here — just flag that it became a real investigation. This creates narrative tension that pays off later in this section. If someone asks a detailed question about it right now, it's fine to say “we'll get into exactly what we found in a few minutes” and keep moving.

⏱ ~2 minutes

SLIDE 22   Four Issues, End to End

**Explain**

Here's the full map of what we're about to walk through. Four issues, in the order we hit them: getting package sourcing genuinely routed through our restricted-network setup, a couple of PostgreSQL quirks, namespace authentication — which took the most work by far — and a 32-bit dependency gap we hit later, during actual report testing.

All four were fully root-caused and resolved. Every single one was investigated with real evidence — logs, direct database queries, configuration screenshots — not guesswork. You'll see that evidence as we go.

**Extra (Optional)**

Worth noting explicitly if the audience is technical: Issue 3, the namespace authentication problem, is structured differently from the other three — it took two full slides just to walk through the evidence, plus a resolution slide, because it was genuinely the hardest problem in this deployment. That's intentional pacing, not padding.

⏱ ~1.5 minutes

SLIDE 23   Issue 1 — Restricted-Network Package Sourcing

**Explain**

First real issue we hit: we'd deliberately chosen a restricted-network posture — no direct internet access from the Cognos server. But when we checked the actual OS package repository configuration, it was still pointed directly at the public AlmaLinux content delivery network. In other words, our security design existed on paper but wasn't actually enforced yet.

The fix was straightforward once identified: we already had a Nexus repository server in this environment from an earlier project, already proxying the OS packages we needed. We just re-pointed the server's package manager configuration at that internal Nexus instance instead — and from that point on, zero direct internet route was needed from the Cognos node for OS packages.

**Extra (Optional)**

This is a good moment to emphasize the discipline point if the audience is security-minded: catching a gap between intended and actual configuration, before it caused an incident, is itself a meaningful engineering practice — worth calling that out explicitly rather than just presenting it as a technical fix. The screenshot shows our actual Nexus repository list, including the pre-existing AlmaLinux proxies we reused.

⏱ ~2 minutes — has a screenshot

SLIDE 24   Issue 2 — PostgreSQL Schema Ownership & Authentication

**Explain**

Two separate PostgreSQL issues bundled into one, because they surfaced close together. First: Content Manager's own connection test failed with a permission-denied error on the schema. Turns out, PostgreSQL versions 15 and newer changed their default behavior — they no longer automatically grant table-creation rights on the default schema to a regular user, only to a superuser. That's a genuine, non-obvious version-specific change that catches a lot of people.

Second, separately: a direct connection test failed with an authentication error specifically over IPv6. Our authentication configuration only had a rule written for IPv4 connections, so anything resolving over IPv6 fell through to a different, stricter default rule and got rejected.

Fixed both explicitly — granted the right schema ownership, and added a matching authentication rule that covers IPv6 as well as IPv4. You can see the successful connection test on the right — two clean green checkmarks, which is what finally confirmed both fixes actually worked.

**Extra (Optional)**

Both of these are genuinely reusable findings beyond just this one deployment — the PostgreSQL 15+ schema permission change specifically catches a lot of people upgrading or building fresh on modern PostgreSQL versions, and the IPv4-only authentication-rule gap is an easy one to miss because `localhost` doesn't always resolve the way people assume it will. Worth stating plainly that these are the kind of details that don't show up until you actually hit them.

⏱ ~2.5 minutes — has a screenshot

SLIDE 25   Issue 3 — Namespace Authentication (Part 1: The Evidence)

**Explain**

This is the one I want to walk through carefully, because it's the most important part of this section — not because it's a failure, but because of what the evidence actually shows, and because of what we ended up doing about it.

Every login attempt against the built-in Cognos namespace failed with the same error, and critically, no login form ever even rendered.

Here's what we systematically ruled out, each with direct evidence, not assumption: the system clock — confirmed correctly synchronized. Database connectivity — the content store connects and builds its schema cleanly, every single time we tried. Schema permissions — fixed and confirmed, as we just saw. The security firewall's domain allow-list — populated correctly. The environment's network address configuration — corrected and verified. And, most tellingly, we did a complete clean reinstall against a completely fresh database, and got the exact same result.

**Extra (Optional)**

If a technical audience wants more specificity: the actual failing error codes were AAA-AUT-0013 paired with CM-CAM-4005 — these are documented IBM error codes, and we cross-referenced official IBM troubleshooting documentation throughout the investigation. Worth mentioning that we consulted IBM's own official docs multiple times during this process — this wasn't guesswork, it was structured troubleshooting against the vendor's own guidance, which simply didn't resolve the built-in namespace specifically in our environment. Stay factual here — the payoff comes in a couple of slides.

⏱ ~2.5 minutes — has a screenshot

SLIDE 26   Issue 3 — Namespace Authentication (Part 2: An Interim Workaround)

**Explain**

With the built-in namespace unresolved after exhausting the troubleshooting we just walked through, we made a scoped, deliberate decision — enable anonymous access temporarily, specifically to unblock functional testing of reporting and dashboards. This is not us quietly hiding an unresolved problem. It's a documented, reversible, temporary decision, and I'm telling you about it directly, right now.

And I want to be clear that this wasn't the end of the story — rather than open a vendor support case and wait, we took one more run at it, from a different angle entirely. I'll show you exactly what we did, and it worked.

**Extra (Optional)**

This is the pivot point of the whole section — the tone should shift here from ‘here's what we couldn't fix’ to ‘here's what we tried next.’ A useful transition line: “knowing exactly where one troubleshooting path reaches its limit is valuable — but so is knowing there's another path worth trying before calling in outside help. That's exactly what we did next.”

⏱ ~2 minutes — land this as a pivot, not a conclusion

SLIDE 27   Issue 3 — Update: Resolved via LDAP

**Explain**

Here's what we did next, instead of stopping at a vendor case. The built-in Cognos namespace has its own internal seeding logic, and that's specifically what was failing. LDAP is architecturally different — Cognos does its own protocol handshake directly against a real directory server, largely independent of whatever was blocking the built-in namespace's internal database seeding.

So we stood up a minimal OpenLDAP directory — sourced through the same Nexus repository pattern we'd used throughout this deployment — seeded a base domain, loaded the schemas OpenLDAP needs to understand standard user attributes, and created one test user. Then we configured a brand new LDAP namespace inside Cognos Configuration, pointing at that directory.

Along the way we hit two more small issues — an admin password lockout from a shell-quoting mistake, and a syntax error from missing directory schemas — both fixed quickly and both worth mentioning briefly, because they're genuinely useful, specific findings, not just noise.

**Extra (Optional)**

If asked why this worked when the built-in namespace didn't: the honest, precise answer is that this result is itself powerful diagnostic evidence — it tells us definitively that the problem was isolated to Cognos's own namespace-seeding logic specifically, not a platform-wide authentication failure, not a network issue, not a certificate problem. That's a much more precise, useful finding than 'authentication doesn't work' — and it's exactly the kind of thing that would make a strong, narrow IBM support case if we ever wanted to pursue why the built-in path specifically fails, now that we know it's isolated to that one code path.

⏱ ~2.5 minutes

SLIDE 28   Issue 3 — Evidence: Cognos Configuration Test Success

**Explain**

Before trusting this in the actual browser, we tested it the right way — inside Cognos Configuration itself. You can see here both a successful namespace connection test and a live logon test against our seeded test user, with the retrieved user profile actually showing on screen — name, username, email, all pulled correctly from the directory.

**Extra (Optional)**

Worth pointing out explicitly if the audience is technical: this two-step verification — test the namespace connection, then separately test an actual logon — is the same disciplined pattern we used for the PostgreSQL content store test earlier in this section. Consistent methodology throughout, not just for the parts that worked easily.

⏱ ~1.5 minutes — has a screenshot

SLIDE 29   Milestone — A Genuine, Non-Anonymous Session

**Explain**

And here's the actual result. On the left, the real Cognos login page — now explicitly presenting our LDAPTest namespace by name. On the right, a genuine authenticated session: signed in as our real test user, session details visible right there in the top corner. Not anonymous. Not a workaround. A real, working, directory-authenticated login.

This means we can now go back and disable anonymous access entirely — the platform has a legitimate, functioning authentication path, and everything from here forward in this presentation, including the live demo, can run on a properly authenticated session rather than the temporary anonymous-access state we described a few minutes ago.

**Extra (Optional)**

This is a genuinely good moment to let the room sit with the result for a second rather than rushing past it — it's the resolution of the single longest, most detailed thread in the entire deployment story. If asked whether the underlying built-in-namespace bug is still unexplained: yes, honestly — we have a working alternative, not a root-cause fix for the built-in path specifically. That's a fair, precise distinction to hold if pressed on it, and it's a better answer than overclaiming full resolution of the original mystery.

⏱ ~2 minutes — has two screenshots

SLIDE 30   Issue 4 — Reports Failed to Run: a 32-Bit Dependency Gap

**Explain**

One more issue, and this one showed up later, once we were actually testing reports rather than just getting the platform to log in. Report execution failed with a generic error — DPR-ERR-2002 — and the Administration console was genuinely misleading here: every service showed as ‘Available,’ and there was no activity logged at all for the failed run, meaning the request never even reached a worker process.

The real answer was buried in the server log, not the UI: the report-rendering engine — a component called BIBusTKServerMain — is shipped as a 32-bit binary, even inside the otherwise-64-bit Linux build of Cognos. Our server was a clean, standard 64-bit-only RHEL install, so it simply didn't have the 32-bit compatibility libraries that binary needs to run at all.

Fixed by installing the matching 32-bit and 64-bit versions of a handful of system libraries together — we hit one extra wrinkle where our proxied package repo was serving a slightly different version than what was already installed, which had to be resolved first. Once that was sorted, the report ran successfully.

**Extra (Optional)**

This is a genuinely valuable, non-obvious finding worth emphasizing on its own merits: it's a documented OS prerequisite gap, not a corrupted download or an installer malfunction — the installer copied every file correctly. Worth mentioning as a concrete recommendation: this is now something we'd add to a pre-installation checklist for any future Cognos Linux deployment, rather than discovering it reactively during report testing. Also worth noting, if asked, that the Administration console's health scorecard alone wasn't enough to catch this — only a real, live test report actually surfaced it, which is a good argument for always running one as part of post-install verification.

⏱ ~2.5 minutes

SECTION 06

Live Capability Demo

Slides 31–37

SLIDE 31   Section divider — Live Capability Demo

**Explain**

Now for the part everyone's actually been waiting for. Everything up to now was context — what the platform is, what it takes to run it, and exactly how our specific deployment went. Now let's see it working, live, on real data.

**Extra (Optional)**

If time is tight, this is the section to protect — trim earlier sections rather than rushing the live demo. A live demo that feels rushed undercuts everything you just spent twenty-plus minutes building credibility for.

⏱ ~15 seconds

SLIDE 32   Milestone — A Working, Authenticated Instance

**Explain**

Before we get into the actual demo flow, I want to pause on this one screenshot, because it's easy to breeze past and it shouldn't be. This is the platform, live, reachable over the network, running on a genuine authenticated session through the LDAP namespace we just stood up — not anonymous access, not a workaround. This is the actual foundation everything else in this demo sits on.

**Extra (Optional)**

Worth an explicit callback here: “this is genuinely the same instance where we hit the package sourcing issue, the PostgreSQL issues, and the authentication issue — all of it. What you're about to see runs on a properly authenticated session, not the anonymous-access state we described a few minutes ago.” That connection matters for credibility — it proves every fix actually held together end-to-end, not just in isolation.

⏱ ~1.5 minutes

SLIDE 33   Demo Step 1 — Upload

**Explain**

We're using a retail sales dataset — transactions, products, regions, dates — uploaded directly through the browser. And right away, Cognos offers a choice worth pointing out: an automatic dashboard, which is fast but skips real modeling, or a proper data module, which builds a governed semantic layer first.

We deliberately chose the data module path. Not because the automatic option is bad — it's genuinely useful for quick self-service exploration — but because building the governed model first tells a stronger story: raw, scattered data becoming a trusted, reusable foundation, which is the actual pitch for this whole platform.

**Extra (Optional)**

Good moment for a live aside if doing this as an actual live demo rather than walking through screenshots: you can literally click back to this upload screen later and choose Automatic Dashboard as a second, contrasting example — “and if you just want something instant, Cognos does that too” — showing both self-service speed and governed rigor in one session is a stronger dual-audience pitch than either alone.

⏱ ~2 minutes

SLIDE 34   Demo Step 2 — Data Module

**Explain**

Here's the data module editor, with our uploaded data already parsed correctly — every column typed appropriately right out of the gate. From here, this is where the real modeling work happens: verifying column types, adding a calculated measure, building a date hierarchy so time-based analysis works automatically later.

This step is genuinely the least visually exciting part of the whole demo, and also the most important — it's the difference between a one-off chart and a trustworthy, reusable foundation other people in the organization can build on too.

**Extra (Optional)**

If you actually built a calculated measure and date hierarchy in your own module before presenting, narrate the specific example — e.g., “we added a profit-margin calculation here, dividing profit by sales, so every report downstream can just reference that measure instead of everyone recalculating it their own way.” A concrete, specific example lands better than describing the capability abstractly.

⏱ ~2.5 minutes

SLIDE 35   Demo Flow Overview

**Explain**

This slide is really a roadmap for what I'm about to click through live: upload, which we've covered; the data module, which we've also covered; a report — a grouped list and a crosstab view, both built from that same trusted model with no re-modeling needed; a dashboard — KPI cards, a revenue trend, a category breakdown, all cross-filtering live from a single click; and finally, AI-assisted natural language querying against that same governed model, which is the current differentiator IBM is pushing hardest with Cognos right now.

**Extra (Optional)**

If your build included the Agentic AI component, this is the moment to actually demonstrate a natural-language query live — something as simple as typing “show me revenue by region” and letting Cognos generate the visualization automatically is a genuinely strong closing beat for the technical portion of the demo, and ties directly back to the TechXchange ‘agentic era’ material referenced in Section 2.

⏱ ~1 minute to introduce, then however long your live click-through actually takes — budget 5–10 minutes for the real demo if doing it live

SLIDE 36   Governance Demo

**Explain**

One more thing worth showing, not just telling: governance, live. We set up a second, restricted user account — not an administrator, a scoped viewer — and demonstrate three things directly. First, two different roles genuinely see two different views of the same platform. Second, content one user creates in a private folder is confirmed invisible to the other — we don't just say permissions work, we show it. And third, a durability check: we stop and restart the Cognos service entirely, and everything — content, permissions, all of it — persists, which proves the underlying database is genuinely durable and not just an in-memory demo trick.

**Extra (Optional)**

This slide is the direct callback to the 'Governance at scale' capability from slide 17 — if you planted the earlier line about proving governance live rather than just describing it, this is where that promise gets paid off. Worth saying explicitly: “remember earlier when I said governance is the one capability we'd actually prove rather than describe? This is that.”

⏱ ~3 minutes

SLIDE 37   Closing — Thank You

**Explain**

That's the full arc — what Cognos Analytics actually is, what it takes to run, the real journey we went through building it, and a live look at what it can actually do. I'd rather leave time for real questions than rush through a summary slide, so let's open it up — technical questions, business questions, anything about the deployment or the platform itself.

**Extra (Optional)**

Have the namespace authentication evidence trail (Slides 25–29) ready to reference quickly if someone asks a pointed follow-up about it — that's the most likely area for detailed technical questions, given how much evidence was presented. Also worth having a one-line answer ready for “what's next” — e.g., opening an IBM support case for the built-in namespace issue specifically, and/or extending the demo to Premium's AI-assisted authoring features in a follow-up session.

⏱ Open-ended — budget at least 10–15 minutes for genuine Q&A given the depth of material covered

