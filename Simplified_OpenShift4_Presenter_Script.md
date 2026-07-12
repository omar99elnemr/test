# Simplified OpenShift 4 Presenter Script

This is a simplified version...

# OpenShift 4 Presentation — Presenter Script & Q&A Prep
### Your cheat sheet for the next 2 hours and the talk itself

---

## HOW TO USE THIS DOCUMENT (read this first — 30 seconds)

You have ~19 slides and limited prep time. Priority order for your remaining time:

1. **Read the "Slide-by-Slide Script" once, out loud if you can** (15–20 min). You don't need to memorize it — just get the *shape* of each slide's story in your head.
2. **Read the "Top 15 Likely Questions" section twice** (15 min). This is your highest-leverage prep — leads almost always ask from this list.
3. **Skim "If You Get Stuck" and "Honesty Phrases"** (5 min) — these are your safety net.
4. **Do one full run-through talking through the slides** using the script as a guide, not a script to read verbatim (20–30 min).
5. Everything else (deep-dive appendix) is for if you have spare time or want to sound sharper on follow-ups.

**You do not need to know everything.** You need to sound calm, structured, and honest about the edges of your knowledge. That combination reads as *more* credible to technical leads than pretending to know everything.

---

## OPENING (memorize this part — first 30 seconds set the tone)

> "Today I want to walk you through OpenShift 4 — Red Hat's enterprise Kubernetes platform. I'll cover why it exists, how it's architected on top of Docker and Kubernetes, and then go through the core workflow: how code becomes a running, secured, network-exposed application. I'll flag the pieces that are OpenShift-specific versus standard Kubernetes as we go, since that distinction tends to be the most useful thing to walk away with."

This framing does three things: sets scope, signals you know the K8s/OpenShift boundary (a real signal of understanding), and gives you permission to be structured rather than encyclopedic.

---

## SLIDE-BY-SLIDE SCRIPT

### Slide 1 — Title
"OpenShift 4 — Red Hat's enterprise Kubernetes platform. Over the next few minutes I'll walk through the architecture and the core developer workflow."

### Slide 2 — Agenda
"Seven areas: why OpenShift exists, the architecture, projects and users, builds and deployments, networking, storage and governance, and security. Let's start with the 'why.'"

### Slide 3 — Why OpenShift
****Main idea:**** Kubernetes is an orchestration engine, not a full platform. It has no opinion on how you build images, where you store them, or how you expose them externally by default.

**Explain:** "Raw Kubernetes gives you scheduling and self-healing — but nothing about builds, registries, or routing. Teams that adopt vanilla Kubernetes usually end up bolting on Jenkins, a registry, an ingress controller, and a homegrown RBAC layer. OpenShift ships all of that pre-integrated on day one."

### Slide 4 — Four Editions
**Explain:** "OpenShift comes in four flavors — Origin is the open-source upstream everything else builds on, Online is Red Hat's hosted version, Dedicated is a managed private cluster on AWS or GCP, and Enterprise is the on-prem version for your own data center. Today's focus is Origin, since it's the technical foundation for all the others."

### Slide 5 — Two Technologies, One Platform
****Main idea:**** This is your most important architectural slide. Get this one solid.

**Explain:** "OpenShift is genuinely three layers. Docker — or more precisely, a container runtime — packages your app into a portable image. Kubernetes orchestrates those containers: scheduling, healing, scaling. OpenShift sits on top and adds everything an enterprise team actually needs day-to-day: CI/CD, a built-in registry, routing, RBAC. The important thing to internalize is that every OpenShift object is a Kubernetes object underneath — Pods, Deployments, Services all exist unchanged. OpenShift *adds* resources like Routes, BuildConfigs, and ImageStreams; it doesn't replace the Kubernetes model."

### Slide 6 — The Toolchain
**Explain:** "Zoom out for a second — any serious dev platform needs six things: source control, a build pipeline, an image registry, networking, an API layer, and governance. OpenShift ships all six integrated, instead of you assembling them from six different vendors."

### Slide 7 — Cluster Architecture
**Explain:** "Under the hood, a cluster has a control plane — API server, etcd, scheduler, controller manager — and worker nodes where your actual application Pods run. This is standard Kubernetes architecture. OpenShift's contribution here is mostly operational: it adds routers, an internal registry, and monitoring as first-class citizens of the cluster rather than optional add-ons."

### Slide 8 — Projects & Users
****Main idea:**** Project = Namespace + extras. This is a very commonly asked distinction.

**Explain:** "A Project is OpenShift's unit of multi-tenancy. Technically it's a Kubernetes Namespace, but OpenShift layers default quotas, role bindings, and network policy on top automatically. Each team gets its own Project — they can't see or touch each other's resources unless a cluster admin explicitly grants access."

### Slide 9 — S2I Build Process
****Main idea:**** This is OpenShift's signature feature. If you remember one build concept, make it this one.

**Explain:** "Source-to-Image, or S2I, is what makes OpenShift's build system distinctive. Instead of requiring every team to write and maintain a Dockerfile, S2I injects your source code directly into a framework-aware builder image — Node, Python, Java, whatever — and produces a runnable image. It's faster because the builder image is pre-optimized, and it's easy to patch: rebuild the base builder image and every app built from it inherits the fix."

### Slide 10 — S2I vs Docker vs Custom
**Explain:** "OpenShift actually supports three build strategies. S2I is the default and most common. Docker Build is for teams that already have Dockerfiles and want exact parity with `docker build`. Custom Build hands you full control via your own builder image — used for edge cases like unusual multi-stage pipelines. Builds can be triggered by a webhook, an image change upstream, a config change, or manually."

### Slide 11 — ImageStreams
****Main idea:**** This trips people up — it's genuinely OpenShift-specific and worth being able to explain simply.

**Explain:** "An ImageStream is OpenShift's abstraction over image tags. A plain Docker registry just stores tagged images — it has no concept of 'this tag changed, go do something.' An ImageStream tracks that change and can automatically trigger a rebuild or redeploy. It also decouples your deployment config from a specific registry URL — you reference the ImageStream, not `docker.io/whatever:latest` directly."

### Slide 12 — DeploymentConfig vs Deployment
****Main idea:**** Very likely question. Know this cold.

**Explain:** "OpenShift originally had its own DeploymentConfig object, which supports OpenShift-specific triggers and lifecycle hooks — pre, mid, post deploy — but isn't portable to plain Kubernetes. Standard Kubernetes Deployments, managed via ReplicaSets, are portable and are what OpenShift 4 defaults new applications to today. DeploymentConfig still exists for legacy workloads or when you need those extra hooks."

### Slide 13 — Services & Routes
****Main idea:**** Route is OpenShift-specific; the Kubernetes equivalent is Ingress. Always mention this pairing.

**Explain:** "A Service gives you a stable internal IP that load-balances across Pods, even as they're replaced. To expose that externally, OpenShift uses a Route — which maps a hostname to a Service and handles TLS termination out of the box. The Kubernetes-native equivalent is Ingress; Routes predate Ingress and are still OpenShift's preferred mechanism."

### Slide 14 — Scaling & HPA
**Explain:** "You can scale manually with a single `oc scale` command, or let the Horizontal Pod Autoscaler watch CPU or memory and scale automatically between a min and max bound you define. Manual scaling is instant but needs a human decision; HPA reacts to real load automatically."

### Slide 15 — Storage, Resources, Quotas
**Explain:** "Three separate but related concepts. Persistent storage: Pods are ephemeral, so anything that needs to survive a restart goes through a PersistentVolumeClaim, either statically pre-provisioned or dynamically created via a StorageClass. Resource requests and limits control how much CPU and memory a Pod can consume — exceed the memory limit and you get OOM-killed; exceed CPU and you just get throttled. ResourceQuota caps the *total* consumption per Project so one team can't starve the cluster."

### Slide 16 — RBAC & SCC
****Main idea:**** This is the #1 "sounds impressive if you get it right" slide. RBAC = who can call the API and do what. SCC = what a Pod is allowed to do at the OS level. They are different layers.

**Explain:** "Two separate security layers. RBAC — Roles and RoleBindings — controls who can call the API and perform which actions, standard Kubernetes concept. Security Context Constraints, or SCCs, are OpenShift-specific and control what a *running Pod* is allowed to do at the OS level: can it run as root, mount the host filesystem, use host networking. The default SCC is 'restricted' — no root, no privilege escalation — and most workloads should never need anything more permissive."

### Slide 17 — ConfigMaps, Secrets, Operators
**Explain:** "ConfigMaps inject non-sensitive configuration into Pods, decoupled from the image itself. Secrets do the same for sensitive data — base64-encoded and access-controlled, though not encrypted by default at that layer without additional setup. Operators are Kubernetes-native applications that encode operational knowledge — they're how OpenShift manages its own core services, and increasingly how third-party software installs and self-heals on the cluster."

### Slide 18 — End-to-End Flow
**Explain:** "Pulling it together: a developer pushes code, that triggers a BuildConfig running S2I, the resulting image lands in an ImageStream, that triggers a DeploymentConfig or Deployment to roll out new Pods, a Service load-balances traffic to them, and a Route exposes it externally. This entire chain can be fully automated after initial setup — a single `git push` can flow all the way to production."

### Slide 19 — Key Takeaways / Close
**Explain:** "Five things worth remembering: OpenShift is Kubernetes plus Docker plus enterprise tooling, pre-integrated. S2I is the build differentiator. Security is layered — RBAC for API actions, SCC for Pod-level OS privileges. Routes replace raw Ingress. And Projects give real multi-tenancy out of the box. Happy to take questions."

---

## TOP 15 LIKELY QUESTIONS (with strong answers)

**1. "What's the actual difference between OpenShift and vanilla Kubernetes?"**
> "Kubernetes is the orchestration engine — scheduling, self-healing, scaling. OpenShift adds everything around it that an enterprise needs to actually ship software on top of that engine: an integrated build system with S2I, a built-in image registry, Routes for external exposure, tighter default security via SCCs, and multi-tenancy via Projects. Every OpenShift object is still a Kubernetes object underneath."

**2. "Why would we use OpenShift instead of just running EKS/GKE/AKS?"**
> "Managed cloud Kubernetes gives you the control plane, but you're still responsible for assembling CI/CD, a registry, ingress, and RBAC policy yourself — or buying separate tools for each. OpenShift's value proposition is that it's opinionated and pre-integrated, plus it runs consistently across on-prem, any cloud, or hybrid — which matters a lot for organizations that don't want to be locked into one cloud vendor's Kubernetes flavor."
*(If you're honest about not knowing pricing/licensing specifics, say so — see Honesty Phrases below.)*

**3. "Is OpenShift open source?"**
> "The upstream project, OpenShift Origin — now branded as OKD — is fully open source. OpenShift Container Platform is Red Hat's supported, commercial distribution of that same codebase, which is what most enterprises actually run in production because of the support SLAs."

**4. "What is S2I and why not just use a Dockerfile?"**
> "S2I builds an image directly from source code using a framework-aware builder image, without requiring a Dockerfile. It's faster to onboard developers who don't want to learn Dockerfile syntax, and patching is centralized — update the builder image and every app rebuilt from it picks up the fix. You can still use a plain Dockerfile via OpenShift's Docker build strategy if you need full control."

**5. "How does OpenShift handle security differently from Kubernetes?"**
> "Two additions on top of standard Kubernetes RBAC. First, Security Context Constraints — SCCs — govern what a Pod can do at the OS level, like running as root or mounting host paths, defaulting to a restrictive policy. Second, Projects come with tighter default isolation and quota enforcement out of the box, so multi-tenant security is less something you have to build yourself."

**6. "What's a Route and how is it different from Ingress?"**
> "A Route is OpenShift's mechanism for exposing a Service externally via a hostname, with built-in TLS termination options. It predates Kubernetes Ingress and is still the preferred mechanism in OpenShift. Functionally they solve the same problem; OpenShift supports both, but Routes are the native, first-class option."

**7. "Can this run on AWS / Azure / GCP / on-prem?"**
> "Yes — that's one of its core selling points. The same OpenShift distribution runs on any major public cloud, on-prem in a data center via OpenShift Enterprise, or as a managed service via OpenShift Dedicated. That portability is a major reason enterprises choose it over a single cloud vendor's native Kubernetes offering."

**8. "How do you handle secrets — are they actually secure?"**
> "Secrets are base64-encoded and access-controlled via RBAC, stored in etcd. Base64 is encoding, not encryption, so out of the box it's obfuscation plus access control rather than true encryption at rest — for stronger guarantees you'd enable etcd encryption at rest or integrate an external secrets manager like Vault. That's a detail I'd want to double check before making a hard claim in a security review."

**9. "What happens if a Pod fails or a node goes down?"**
> "That's standard Kubernetes self-healing, which OpenShift inherits fully. The controller manager continuously reconciles actual state against desired state — if a Pod dies, its controller (ReplicaSet or DeploymentConfig) creates a replacement. If a whole node goes down, the scheduler reschedules affected Pods onto healthy nodes, assuming there's capacity."

**10. "How does scaling work — is it automatic?"**
> "Both. You can scale manually with a single command for predictable load, or configure a Horizontal Pod Autoscaler to watch CPU or memory metrics and scale automatically between a min and max replica count you define."

**11. "What's the learning curve like for a team coming from plain Kubernetes?"**
> "Pretty shallow for the core model, since Pods, Services, and Deployments work identically. The learning curve is mostly around the OpenShift-specific additions — S2I builds, ImageStreams, Routes, and SCCs — which are additive concepts, not replacements for Kubernetes knowledge."

**12. "Does OpenShift support GitOps / ArgoCD?"**
> "Yes, OpenShift has strong first-class support for GitOps workflows, typically via the OpenShift GitOps Operator, which is Red Hat's supported distribution of Argo CD. I haven't personally run that setup yet, but it's a well-documented, common pattern for teams that want declarative, Git-driven deployments." *(Honest framing — see below.)*

**13. "What about cost — is OpenShift expensive?"**
> "I don't have current pricing details to give you an accurate number — that's something I'd want to confirm with Red Hat directly or check their current documentation, since subscription models change. Broadly it's licensed per-node/per-core through Red Hat subscriptions, layered on top of whatever infrastructure cost you're already paying."

**14. "How is monitoring/observability handled?"**
> "OpenShift ships with built-in cluster monitoring based on Prometheus and Grafana, deployed and managed via an Operator by default. That's an area I've only covered at a high level — happy to go deeper on the monitoring stack in a follow-up if that's a priority for your use case."

**15. "What's an Operator, really — in plain terms?"**
> "Think of it as encoding a human operator's runbook into software. A normal Kubernetes Deployment just keeps a fixed number of Pods running. An Operator understands the *application-specific* operational knowledge — how to safely upgrade a database, how to fail over, how to back it up — and automates that. OpenShift itself is largely self-managed by Operators for its core services like networking and monitoring."

---

## IF YOU GET A QUESTION YOU DON'T KNOW (honesty phrases — memorize 2-3 of these)

Technical leads respect calibrated confidence far more than false certainty. Use phrases like:

- *"That's a good question — I haven't gone deep on that specific area yet, but here's what I understand at a high level: [give the closest adjacent thing you do know]. I'd want to verify the specifics before committing to an exact answer."*
- *"I'm not fully confident on the exact mechanism there — my understanding is [X], but I'd want to confirm that against the docs before you rely on it."*
- *"That's on my list to go deeper on — right now I know it exists and roughly what problem it solves, but not the implementation details yet."*
- *"Great follow-up — I'll be honest, that's past what I've covered so far. Can I follow up with you after I've confirmed?"*

**Never bluff a specific number, command syntax, or "it definitely works like X" claim you're not sure of.** A wrong confident answer costs you more credibility than an honest "let me confirm that."

---

## IF YOU GET STUCK MID-ANSWER

- Buy time honestly: *"Let me think through that for a second..."* — pausing is fine.
- Redirect to what you do know: *"I can't speak precisely to [X], but the related piece I do know is [Y], which might be part of what you're after."*
- If totally lost: *"I want to give you an accurate answer rather than guess — let me follow up on that one specifically."*

---

## QUICK-REFERENCE GLOSSARY (skim once before you go in)

| Term | One-line definition |
|---|---|
| **Pod** | Smallest deployable unit — one or more containers sharing network/storage |
| **Project** | OpenShift's Namespace + quotas + default RBAC + network policy |
| **BuildConfig** | Defines how to build an image (S2I, Docker, or Custom strategy) |
| **ImageStream** | OpenShift abstraction tracking changes to image tags; can trigger builds/deploys |
| **DeploymentConfig (DC)** | OpenShift-native deployment object with extra triggers/hooks; legacy-leaning now |
| **Deployment** | Standard Kubernetes deployment object, managed via ReplicaSets; now the default |
| **Service** | Stable internal virtual IP load-balancing across matching Pods |
| **Route** | OpenShift object exposing a Service externally via hostname + TLS (≈ Ingress) |
| **PV / PVC** | PersistentVolume (actual storage) / PersistentVolumeClaim (a Pod's request for it) |
| **StorageClass** | Enables dynamic provisioning of PVs on demand |
| **ResourceQuota** | Caps total resource consumption (CPU, memory, object counts) per Project |
| **Role / RoleBinding** | RBAC — defines permitted actions, then binds them to a user/group/SA |
| **SCC** | Security Context Constraint — governs Pod-level OS privileges (root, host mounts, etc.) |
| **ConfigMap** | Non-sensitive config injected into Pods, decoupled from the image |
| **Secret** | Sensitive config (passwords, tokens, certs), base64-encoded, access-controlled |
| **Operator** | Kubernetes-native app encoding operational knowledge for automated management |
| **S2I** | Source-to-Image — builds a runnable image directly from source, no Dockerfile |
| **CRC** | Code Ready Containers / OpenShift Local — single-node cluster on your laptop |

---

## APPENDIX: DEEPER NOTES (only if you have spare minutes)

**On DeploymentConfig triggers specifically:**
DCs support three trigger types — `ConfigChange` (redeploy when the DC's pod template changes), `ImageChange` (redeploy when a watched ImageStream tag updates), and manual. Kubernetes Deployments only react to changes in their own pod spec — they don't have a native "watch this external image tag" trigger the way DCs do; that gap is usually filled by CI/CD tooling instead.

**On build strategies, one level deeper:**
- **S2I**: two-stage under the hood — assemble script copies source into the builder image and runs a build script; the result is committed as a new image layer.
- **Docker strategy**: OpenShift runs an actual `docker build`/buildah process against your Dockerfile inside the cluster.
- **Custom strategy**: you supply your own builder image implementing the OpenShift build API contract — used for things like producing non-container artifacts or highly specialized pipelines.

**On networking, one level deeper:**
OpenShift's default SDN has historically been Open vSwitch-based; OpenShift 4 has been transitioning toward OVN-Kubernetes as the default CNI plugin. If asked specifically which one, it's safer to say: "OpenShift 4's networking has moved toward OVN-Kubernetes as the default CNI in recent versions — I'd want to confirm the exact version cutover" rather than commit to a specific version number from memory.

**On the control plane, one level deeper:**
In OpenShift 4, cluster lifecycle (installation, upgrades, and much of the control plane itself) is managed by the Cluster Version Operator (CVO), which in turn manages a set of second-level Operators for networking, DNS, ingress, etc. This "Operators all the way down" design is a defining trait of OpenShift 4 specifically (versus OpenShift 3, which was more manually managed) — a good detail to drop if someone asks "what's actually new in version 4."

**On multi-tenancy, one level deeper:**
Project isolation by default is primarily RBAC + separate namespace scoping — *network* isolation between projects is not automatic unless NetworkPolicies (or a multitenant SDN plugin) are explicitly configured. Worth knowing so you don't overstate isolation guarantees if pressed.

---

## FINAL 5-MINUTE CHECKLIST BEFORE YOU PRESENT

- [ ] Re-read the Opening line once out loud
- [ ] Skim the Top 15 Q&A one more time
- [ ] Have this doc open on a second screen or printed
- [ ] Remember: structure + honesty > perfect recall
- [ ] Breathe. You clearly understand the architecture well enough to teach it — that's the bar.

Good luck.
