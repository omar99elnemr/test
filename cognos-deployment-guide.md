# IBM Cognos Analytics — Self-Hosted Deployment Guide

Full walkthrough of deploying IBM Cognos Analytics 12.1.3 on AlmaLinux 9, in a restricted network, with a PostgreSQL content store and LDAP authentication. Written start to finish from an actual lab build — every issue below was hit for real, root-caused, and fixed.

**Companion docs:** [`cognos-analytics-deployment-plan.md`](./cognos-analytics-deployment-plan.md) (the original decision log) · Instana self-hosted guides (same lab, same conventions)

---

## Table of Contents

- [1. Environment Summary](#1-environment-summary)
- [2. Key Decisions at a Glance](#2-key-decisions-at-a-glance)
- [3. Pre-Installation Checklist](#3-pre-installation-checklist)
- [4. Sourcing the Installer](#4-sourcing-the-installer)
- [5. Installing Cognos Analytics](#5-installing-cognos-analytics)
- [6. PostgreSQL Content Store Setup](#6-postgresql-content-store-setup)
- [7. IBM Cognos Configuration](#7-ibm-cognos-configuration)
- [8. Starting the Service & First Login](#8-starting-the-service--first-login)
- [9. Fixing Authentication with LDAP](#9-fixing-authentication-with-ldap)
- [10. Verification & Test Plan](#10-verification--test-plan)
- [11. Troubleshooting Index](#11-troubleshooting-index)
- [12. Appendix — Quick Reference](#12-appendix--quick-reference)

---

## 1. Environment Summary

| Item | Value |
|---|---|
| Host | `cognos-0.fpclab.local` |
| OS | AlmaLinux 9 / RHEL 9.2 |
| IP | `192.168.2.91` |
| CPU / RAM | 8 vCPU / 32 GB (meets IBM minimum, no headroom) |
| Disk | 70 GB total |
| Cognos version | 12.1.3, Custom Install, single-server |
| Content store | PostgreSQL 16 |
| Authentication | LDAP (OpenLDAP, local) |
| Package source | Internal Nexus (`192.168.2.90:8081`) — no direct internet from `cognos-0` |

> 📌 **NOTE**
> This is a **single-server** topology — Content tier, Application tier, and Agentic AI all on one VM, no Gateway. IBM documents single-server as the standard approach for demonstration and proof-of-concept environments, which is exactly what this lab is.

---

## 2. Key Decisions at a Glance

Every decision below was made against real alternatives, not defaulted into. Full reasoning lives in the companion deployment plan doc — this is the summary.

| Decision point | Chosen | Why (short version) |
|---|---|---|
| OS | AlmaLinux 9 | Matches lab fleet; Easy Install (the Windows-only path) is flagged by IBM as not recommended for production |
| Topology | Single-server | IBM's documented standard for demo/PoC environments |
| Network | No direct internet; Nexus-proxied | Real least-privilege security posture, reuses proven Nexus pattern |
| Content store | PostgreSQL 16 | No licensing, open-source, consistent with rest of lab |
| Gateway | None | Application tier serves gateway functions unless Kerberos SSO is needed (not in scope) |
| Authentication | LDAP (after the built-in namespace failed and multiple culprits were ruled out) | See [Section 9](#9-fixing-authentication-with-ldap) — this took real troubleshooting |

---

## 3. Pre-Installation Checklist

- [ ] Confirm VM meets minimums — see table below
- [ ] Set hostname and static IP
- [ ] `ulimit -n 8192` and persist via `/etc/security/limits.conf`
- [ ] Point `/etc/yum.repos.d/` at your internal Nexus proxy (not public repos)
- [ ] Firewall: `firewalld` **enabled**, not disabled — open only the ports you actually need (Section 5.4)
- [ ] Confirm disk space — see disk table below
- [ ] Stage installer files and JDBC driver in Nexus's hosted repo

### Hardware requirements — minimum / safe / recommended

| Resource | IBM minimum | Safe minimum | Recommended |
|---|---|---|---|
| CPU | 4 cores | 4–6 cores | 8+ cores |
| RAM | 32 GB | 32 GB | 32 GB+ (32 GB has **no headroom**) |
| Disk (install) | 18 GB | 40–50 GB | 100 GB+ |
| Disk (temp dir) | 8 GB | 15–20 GB | Report rendering uses this heavily |
| File descriptors | 8192 | 8192 | 8192, set explicitly |

> 💡 **TIP**
> The temp directory matters more than people expect. Every report render — regardless of output format — goes through temp space first. Running low causes failures that look completely unrelated to disk space unless you know to check it.

---

## 4. Sourcing the Installer

### 4.1 Getting the right media from Passport Advantage

Cognos Analytics Server ships as **two files that must sit together** — an installer framework and a payload zip:

| File | Purpose |
|---|---|
| `analytics-installer-5.0.18.bin` | IBM's shared installer framework (the launcher) |
| `ca_srv_lnx86_12.1.3.zip` | Cognos Analytics Server 12.1.3, Linux x86 — the actual payload |

> ⚠️ **WARNING — easy to pick the wrong media**
> Passport Advantage lists many similarly-named eAssemblies. Ruled out during this deployment: Linux on System z / System p LE (wrong CPU architecture), AIX/Windows builds (wrong OS), Cognos Controller (different product), SPSS add-ons (plug-ins for an existing install, not the base server), Certified Containers builds (container-specific, not a bare-metal install), Cognos PowerPlay/Transformer (legacy/companion products), and the **Analytics Administrator** license tier (restricted authoring — you need **Advanced** if you plan to build reports/dashboards, not just administer).

> 📌 **NOTE**
> **Do not manually extract the `.zip`.** The installer consumes it directly and handles extraction internally. An earlier manual-extraction attempt in this lab produced a loose folder of Java library directories that wasn't usable — the clean zip had to be used instead.

### 4.2 Staging via Nexus (restricted network)

Since `cognos-0` has no direct internet access, both files are staged through the lab's internal Nexus instance:

```bash
mkdir -p /opt/cognos-install
cd /opt/cognos-install
# upload both files to Nexus's cognos-hosted repo first (from a machine with internet access), then:
curl -O http://192.168.2.90:8081/repository/cognos-hosted/analytics-installer-5.0.18.bin
curl -O http://192.168.2.90:8081/repository/cognos-hosted/ca_srv_lnx86_12.1.3.zip
chmod +x analytics-installer-5.0.18.bin
```

`![Screenshot placeholder: nexus repository list showing cognos-hosted and existing proxies](./images/nexus_repo_list.png)`

---

## 5. Installing Cognos Analytics

### 5.2 Run the installer — requires a GUI session and sudo

> 📌 **NOTE — two hard requirements before you start**
> - **A local GUI desktop session.** This installer is Java Swing-based and needs a display to render — run it from the VM's own desktop session (console or a remote desktop protocol that gives you a real display), not a plain SSH terminal.
> - **`sudo` privileges.** The install target sits under `/opt`, which is root-owned by default — run the installer with `sudo` so it can write to its target directory without a permissions error.

```bash
cd /opt/cognos-install
sudo ./analytics-installer-5.0.18.bin -DREPO=./ca_srv_lnx86_12.1.3.zip -r ./response.properties
```

> 💡 **TIP**
> `-DREPO=<path>` tells the installer where the payload zip is — it's **not auto-detected**, even with the `.bin` and `.zip` sitting in the same directory. `-r <path>` captures a response file as you click through the wizard (click to the summary panel, then cancel — you don't need to complete a real install just to generate this file), which you can reuse later for a scripted, repeatable silent install with `-f <path>`.

### 5.3 Choose components

At the **Component Selection** screen, hold **Ctrl** and click to select multiple:

![Component selection screen — Content tier, Application tier, Agentic AI, Gateway, Image service options](./images/component_selection.png)

| Component | Include? | Why |
|---|---|---|
| Content tier | ✅ Yes | Required — connects to your content store |
| Application tier | ✅ Yes | Required — reporting, dashboards, query services |
| Agentic AI | Optional | Include if you plan to demo AI-assisted reporting/natural-language querying |
| Gateway | ❌ No | Application tier already serves this role unless you need Kerberos SSO or multi-node load balancing |
| Image service | ❌ No | Only needed for map/tile visualizations; requires Docker and outbound internet — conflicts with a restricted-network design |

> 💡 **TIP**
> Deciding on Agentic AI now matters — skipping it means redoing the install later if you want to demo it. If your presentation/demo plan includes AI-assisted reporting, select it now.

### 5.4 Firewall — before starting the service

```bash
sudo systemctl enable --now firewalld
sudo firewall-cmd --permanent --add-port=9300/tcp    # Dispatcher / web UI
sudo firewall-cmd --permanent --add-port=9301/tcp    # Dataset Service
sudo firewall-cmd --permanent --add-port=9362/tcp    # Local log server
sudo firewall-cmd --permanent --add-port=9303-9323/tcp  # NodeJS backend services
sudo firewall-cmd --reload
sudo firewall-cmd --list-ports
```

> 📌 **NOTE**
> Outbound traffic (e.g., to Nexus) does **not** need an explicit firewalld rule — firewalld only filters inbound connections by default; outbound is permissive unless you've deliberately configured a stricter default-deny-outbound posture. Confirmed and deliberately left as-is in this lab.

---

## 6. PostgreSQL Content Store Setup

### 6.1 Install via Nexus-proxied repos

```bash
sudo dnf module list postgresql
sudo dnf module enable postgresql:16 -y
sudo dnf install -y postgresql-server postgresql-contrib
sudo postgresql-setup --initdb
sudo systemctl enable --now postgresql
```

> 📌 **NOTE**
> Confirm `/etc/yum.repos.d/` is pointed at your Nexus proxy **before** this step — otherwise `dnf` will try to reach the public internet directly, defeating the restricted-network design. Check with `dnf repolist`.

### 6.2 Create the database and user

```bash
sudo -u postgres psql
```
```sql
CREATE DATABASE cognoscs WITH ENCODING 'UTF8';
CREATE USER cognos_svc WITH PASSWORD 'your_strong_password_here';
GRANT ALL PRIVILEGES ON DATABASE cognoscs TO cognos_svc;
```

> ⚠️ **WARNING — PostgreSQL 15+ schema permission change**
> `GRANT ALL PRIVILEGES ON DATABASE` does **not** grant `CREATE` on the `public` schema by default on PostgreSQL 15 and newer (a deliberate security hardening change). Content Manager will fail its connection test with `permission denied for schema public` unless you fix this explicitly:
> ```bash
> psql -h 127.0.0.1 -U cognos_svc -d cognoscs
> ```
> ```sql
> ALTER SCHEMA public OWNER TO cognos_svc;
> GRANT ALL ON SCHEMA public TO cognos_svc;
> GRANT CREATE ON SCHEMA public TO cognos_svc;
> ```

### 6.3 Fix IPv4/IPv6 authentication rules

```bash
sudo nano /var/lib/pgsql/data/pg_hba.conf
```
Add (above any general `ident` rule for the same address):
```
host    cognoscs    cognos_svc    127.0.0.1/32    scram-sha-256
host    cognoscs    cognos_svc    ::1/128         scram-sha-256
```
```bash
sudo systemctl restart postgresql
```

> ⚠️ **WARNING — `localhost` may resolve to IPv6 first**
> A connection test can fail with `Ident authentication failed` even after adding the IPv4 rule above, if `localhost` resolves to `::1` and only the IPv4 line was added. Add **both** lines, or test explicitly against `127.0.0.1`:
> ```bash
> psql -h 127.0.0.1 -U cognos_svc -d cognoscs -W
> ```

### 6.4 JDBC driver

Cognos Analytics 12.1.x runs on a **bundled IBM Semeru 17 JRE**, not your host's system Java — driver compatibility should reference the bundled JRE.

```bash
# Java 8 / JDBC 4.2 driver — correct choice, covers Java 8 through 17+
curl -O https://jdbc.postgresql.org/download/postgresql-42.7.13.jar
# stage via Nexus, then on cognos-0:
cp postgresql-42.7.13.jar /opt/ibm/cognos/analytics/drivers/
```

> 📌 **NOTE**
> If you ever fully reinstall Cognos (`rm -rf /opt/ibm/cognos/analytics`), the JDBC driver is **wiped** along with everything else — it's not part of the installer payload, so it must be manually re-staged into `drivers/` after any fresh install.

---

## 7. IBM Cognos Configuration

### 7.1 Launch

Same requirements as the installer — a local GUI session and `sudo`:

```bash
cd /opt/ibm/cognos/analytics/bin64
sudo ./cogconfig.sh
```

### 7.2 Content Manager → database resource

**Data Access → Content Manager** → right-click → **New resource → Database**, Type = **PostgreSQL database**.

![Content Manager database resource properties](./images/cm_db_resource.png)

| Field | Value |
|---|---|
| Database server and port | `localhost:5432` |
| User ID and password | `cognos_svc` / your password |
| Database name | `cognoscs` |
| Schema name | `public` |
| SSL Encryption Enabled | `False` (fine for a lab) |

> ⚠️ **WARNING**
> **Schema name is required** — Cognos Configuration flags it with a red error icon if left blank. Set it to `public` (or your actual schema) before testing.

![Schema name field, showing the required-field error indicator](./images/cm_schema_field.png)

Right-click the resource → **Test**. You should see two green checkmarks:

![Content Manager database connection test — confirmed success](./images/cm_test_success.png)

### 7.3 Security → Authentication

Set **Allow Anonymous Access = False**.

![Authentication component properties](./images/auth_settings.png)

The built-in **Cognos** namespace is present by default and requires no extra configuration to select. In this lab it was tried first — see [Section 9](#9-fixing-authentication-with-ldap) for what happened when we tried to actually log in with it, and why the deployment moved to LDAP instead.

### 7.4 IBM Cognos Application Firewall (CAF)

**Security → IBM Cognos Application Firewall** — leave **Enable CAF validation? = True** (IBM: *"Under normal circumstances, do not disable"*). Add your server's IP to **Valid domains or hosts**.

![CAF settings](./images/caf_settings.png)

### 7.5 Environment URIs

**Environment** — set every URI (Gateway, Dispatcher, Content Manager) to your server's **IP address**, not its hostname, unless you have DNS resolving that hostname for every machine that will access the UI.

![Environment URIs corrected to the VM's IP](./images/environment_uris.png)

> ⚠️ **WARNING — check for duplicate/stale entries**
> The **Content Manager URIs** field can hold multiple entries. A leftover hostname-based entry alongside a corrected IP-based one caused confusion in this lab — remove any stale entry so only the correct one remains:

![Content Manager URIs field showing two entries — a stale hostname-based one and the corrected IP-based one](./images/cm_uris_dual.png)

**Save** (File → Save) after every section above, and **Test** the full configuration before starting the service.

---

## 8. Starting the Service & First Login

```bash
# In Cognos Configuration: Actions → Start
```

Watch the log live in a separate terminal while it starts:
```bash
tail -f /opt/ibm/cognos/analytics/logs/cognosserver.log
```

Once started:
```bash
curl http://<your-ip>:9300/p2pd/servlet
```
Then browse to `http://<your-ip>:9300/bi/`.

> 💡 **TIP**
> If the login page shows **"AAA-AUT-0013 The user is already authenticated in all available namespaces"** with no login form at all, try a fresh incognito window first — it's often a stale session cookie from before anonymous access was disabled. If that doesn't clear it, see [Section 11](#11-troubleshooting-index).

---

## 9. Fixing Authentication with LDAP

This lab hit a genuinely difficult authentication problem — full detail in [Section 11](#11-troubleshooting-index) — where the built-in Cognos namespace failed to authenticate any user, with no login form ever rendering, across a clean reinstall and a fresh database. After exhausting standard configuration-level troubleshooting, the fix was to configure LDAP instead — a completely independent authentication code path in Cognos.

### 9.1 Install OpenLDAP

```bash
sudo dnf install -y openldap-servers openldap-clients
sudo systemctl enable --now slapd
sudo systemctl status slapd
```

![slapd service started and running](./images/slapd_started.png)

### 9.2 Set the admin password

```bash
slappasswd
```

![slappasswd generating a hashed password](./images/slappasswd_hash.png)

> ⚠️ **WARNING — plaintext vs. hash confusion**
> The `{SSHA}...` output from `slappasswd` is what gets **stored**; you always authenticate later with the **plaintext** password, which LDAP hashes internally to compare. If plaintext auth is rejected right after setting this, the hash itself was likely stored incorrectly — usually a shell quoting issue.
>
> **Use single-quoted heredocs** to avoid the shell interpreting special characters in the hash:
> ```bash
> cat <<'EOF' > /tmp/chrootpw.ldif
> dn: olcDatabase={2}mdb,cn=config
> changetype: modify
> replace: olcRootPW
> olcRootPW: {SSHA}paste_your_exact_hash_here
> EOF
> sudo ldapmodify -Y EXTERNAL -H ldapi:/// -f /tmp/chrootpw.ldif
> ```

![ldapmodify applying the corrected password via the local socket](./images/ldapmodify_success.png)

Verify:
```bash
ldapwhoami -x -D "cn=admin,dc=fpclab,dc=local" -W
```
Should return `dn:cn=admin,dc=fpclab,dc=local`.

### 9.3 Set the domain suffix

```bash
cat <<'EOF' > /tmp/chdomain.ldif
dn: olcDatabase={2}mdb,cn=config
changetype: modify
replace: olcSuffix
olcSuffix: dc=fpclab,dc=local

dn: olcDatabase={2}mdb,cn=config
changetype: modify
replace: olcRootDN
olcRootDN: cn=admin,dc=fpclab,dc=local
EOF
sudo ldapmodify -Y EXTERNAL -H ldapi:/// -f /tmp/chdomain.ldif
```

### 9.4 Load required schemas

> ⚠️ **WARNING — missing schemas cause `Invalid syntax (21)`**
> OpenLDAP ships with minimal core schemas. Adding a user with `objectClass: inetOrgPerson` before loading the schemas that define it fails with:
> ```
> ldap_add: Invalid syntax (21)
> additional info: objectClass: value #0 invalid per syntax
> ```

Fix — load the schemas first:
```bash
sudo ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/cosine.ldif
sudo ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/inetorgperson.ldif
```

### 9.5 Create the base structure and a test user

```bash
cat <<'EOF' > /tmp/base.ldif
dn: dc=fpclab,dc=local
objectClass: top
objectClass: dcObject
objectClass: organization
o: FPCLab
dc: fpclab

dn: cn=admin,dc=fpclab,dc=local
objectClass: organizationalRole
cn: admin

dn: ou=People,dc=fpclab,dc=local
objectClass: organizationalUnit
ou: People
EOF
sudo ldapadd -x -D "cn=admin,dc=fpclab,dc=local" -W -f /tmp/base.ldif
```

```bash
cat <<'EOF' > /tmp/user.ldif
dn: uid=cognostest,ou=People,dc=fpclab,dc=local
objectClass: top
objectClass: inetOrgPerson
uid: cognostest
cn: Cognos Test User
sn: TestUser
userPassword: your_test_password
mail: cognostest@fpclab.local
EOF
sudo ldapadd -x -D "cn=admin,dc=fpclab,dc=local" -W -f /tmp/user.ldif
```

> 💡 **TIP**
> If `ldapadd` fails again with a syntax error, check for hidden characters from a copy-pasted heredoc:
> ```bash
> cat -A /tmp/user.ldif | head -5
> ```
> Look for `^M` (carriage return) at line ends — a common artifact when heredocs are pasted through certain terminal sessions.

Verify:
```bash
ldapsearch -x -D "cn=admin,dc=fpclab,dc=local" -W -b "ou=People,dc=fpclab,dc=local" "(uid=cognostest)"
```

### 9.6 Firewall

```bash
sudo firewall-cmd --permanent --add-port=389/tcp
sudo firewall-cmd --reload
```

### 9.7 Configure the LDAP namespace in Cognos Configuration

**Security → Authentication** → right-click → **New resource → Namespace** → Type(Group) = **LDAP**, Type = **LDAP - General default values**.

![New LDAP namespace resource dialog](./images/ldap_new_namespace.png)

| Field | Value |
|---|---|
| Namespace ID | `LDAPTest` |
| Host and port | `localhost:389` |
| Base Distinguished Name | `dc=fpclab,dc=local` |
| User lookup | `(uid=${userID})` |
| Use bind credentials for search? | `True` |
| Bind user DN and password | `cn=admin,dc=fpclab,dc=local` / plaintext password |

![Bind user DN and password dialog](./images/ldap_bind_creds.png)

Save, then right-click the namespace → **Test**:

![Test logon prompt for the LDAPTest namespace](./images/ldap_test_login_prompt.png)

A successful test shows both the namespace connection and the logon test passing, with the retrieved user profile visible:

![LDAP namespace test success — connection and logon both passing, user profile retrieved](./images/ldap_test_success.png)

### 9.8 Confirm at the real login page

![Login page presenting the LDAPTest namespace](./images/ldap_login_page.png)

![Signed in as the real LDAP-authenticated test user](./images/ldap_logged_in.png)

> ✅ **Result:** genuine authenticated session, not anonymous access. You can now safely re-disable anonymous access (**Security → Authentication → Allow Anonymous Access = False**) if it was left on during earlier testing.

> 📌 **NOTE — what this does and doesn't prove**
> This confirms the built-in namespace's failure was isolated to Cognos's own internal namespace-seeding logic, not a platform-wide authentication or network problem — LDAP uses a completely independent code path. It does **not** explain *why* the built-in namespace specifically failed to seed; that remains open and would be worth an IBM support case if you need the built-in namespace specifically resolved rather than an LDAP-based alternative.

---

## 10. Verification & Test Plan

Once logged in, validate end-to-end functionality:

1. **Upload a dataset** (CSV) — [Team content] → **Upload and create**
2. **Choose Data module** (not Automatic dashboard) for a governed model — verify column types, add a calculated measure, build a date hierarchy

   ![Upload and create screen](./images/upload_create.png)

   ![Data Module editor grid view](./images/data_module_grid.png)

3. **Build a report** — a list report and a crosstab from the same data module
4. **Build a dashboard** — KPI cards, a trend chart, a category breakdown, with cross-filtering
5. **Governance check** — create a second, restricted user; confirm private content stays private
6. **Durability check** — stop/start the Cognos service; confirm content and permissions persist

### 10.1 Report execution failure — `DPR-ERR-2002`

Report execution failed in the browser with a generic error:
```
DPR-ERR-2002 Unable to execute the request because there were no connections to the
process available within the configured time limit.
```

> ⚠️ **WARNING — the UI error is generic; the real cause is in the server log**
> The Cognos Administration console's System status showed every listed service as "Available," with only the parent dispatcher showing "Partially available" — easy to miss. **Current Activities showed no logged activity at all** for the failed run, meaning the request never reached a worker process. The actual error only surfaced in `cogserver.log`:
> ```
> java.io.IOException: Cannot run program "/opt/ibm/cognos/analytics/bin/BIBusTKServerMain":
> Exec failed, error: 2 (No such file or directory)
> ```

**Root cause:** `BIBusTKServerMain` — the report-rendering engine — ships as a **32-bit ELF binary**, even in the otherwise-64-bit Linux x86 build of Cognos Analytics 12.1.3. The file existed on disk with correct ownership and permissions (ruling out a missing-file or permissions problem), but RHEL 9.2 was installed 64-bit-only and had none of the 32-bit compatibility libraries this binary needs to run at all. "No such file or directory" in this context actually meant the binary's **required interpreter** (`/lib/ld-linux.so.2`) was missing — not the binary itself.

**Diagnosis steps, in order:**
1. Checked Cognos Administration → System status — services showed "Available," dispatcher showed "Partially available"
2. Checked Current Activities — nothing logged, meaning the request never reached a worker
3. Checked `cogserver.log` directly — surfaced the real Java exception above
4. Verified the file existed with correct ownership/permissions (`ls -la`) — ruled out missing-file/permissions
5. Checked the binary type — `file BIBusTKServerMain` confirmed 32-bit ELF, requiring `/lib/ld-linux.so.2`
6. Confirmed the interpreter was genuinely absent (`ldd`, `ls /lib/ld-linux.so.2`)

**Fix:**
```bash
# install matching 32-bit + 64-bit compatibility libraries together
dnf install -y glibc.i686 glibc.x86_64 \
  libstdc++.i686 libstdc++.x86_64 \
  ncurses-libs.i686 ncurses-libs.x86_64
```

> ⚠️ **WARNING — multilib version mismatch**
> Installing just the `.i686` packages hit a version-conflict error, because the Nexus-proxied repo was serving AlmaLinux-built packages at a different version than the RHEL-native 64-bit packages already installed. Multilib (32-bit + 64-bit side by side) requires **matching versions across both architectures** — resolved by installing both architectures explicitly together, as shown above, rather than the 32-bit packages alone.

Re-checking the interpreter afterward revealed one further genuinely missing dependency:
```bash
ldd /opt/ibm/cognos/analytics/bin/BIBusTKServerMain
# → libnsl.so.2 => not found
dnf install -y libnsl.i686 libnsl2.i686
```

Verified clean with the same library search path Cognos's own dispatcher uses to launch this process:
```bash
LD_LIBRARY_PATH=/opt/ibm/cognos/analytics/bin ldd /opt/ibm/cognos/analytics/bin/BIBusTKServerMain
```
All dependencies resolved. Restarted Cognos (`cogconfig.sh` → Actions → Stop, then Start), re-ran the report — succeeded.

> 💡 **TIP — add this to your pre-installation checklist next time**
> Install these 32-bit compatibility packages **before** running the Cognos installer at all, rather than reactively:
> ```bash
> dnf install -y glibc.i686 glibc.x86_64 \
>   libstdc++.i686 libstdc++.x86_64 \
>   ncurses-libs.i686 ncurses-libs.x86_64 \
>   libnsl.i686 libnsl2.i686
> ```
> Also worth doing proactively after any install, before going live — scan for other 32-bit binaries in the install tree and confirm each one's dependency chain resolves:
> ```bash
> find /opt/ibm/cognos/analytics/bin -type f -exec file {} \; | grep "32-bit"
> LD_LIBRARY_PATH=/opt/ibm/cognos/analytics/bin ldd <binary>
> ```
> And don't trust the Administration console's service scorecard alone to confirm health — Content Manager and similar services can show "Available" even when the report-rendering engine specifically can't start. Always run a real test report as part of post-install verification, not just a green-light check.

---

## 11. Troubleshooting Index

Quick-reference for the issues actually hit in this deployment, in the order encountered.

| # | Issue | Root Cause | Fix |
|---|---|---|---|
| 1 | `permission denied for schema public` | PostgreSQL 15+ doesn't grant `CREATE` on `public` by default | `ALTER SCHEMA public OWNER TO cognos_svc` |
| 2 | `Ident authentication failed` on `::1` | `pg_hba.conf` only had a rule for IPv4 loopback | Add matching `scram-sha-256` rule for `::1/128` |
| 3 | JDBC driver missing after reinstall | Driver isn't part of the installer payload; wiped on `rm -rf` | Re-copy driver into `drivers/` after any fresh install |
| 4 | `Invalid syntax (21)` adding LDAP user | Missing `cosine`/`inetorgperson` schemas in OpenLDAP | Load both schemas before adding users |
| 5 | LDAP admin plaintext password rejected | SSHA hash corrupted by shell interpreting special characters | Use single-quoted heredocs (`<<'EOF'`) |
| 6 | **AAA-AUT-0013 / CM-CAM-4005**, no login form ever renders | Built-in Cognos namespace's `cmnamespaces` table never seeds — confirmed empty across a clean reinstall and fresh database, despite healthy schema and all other config verified correct | **Not resolved for the built-in namespace itself.** Worked around by configuring LDAP instead ([Section 9](#9-fixing-authentication-with-ldap)) — a genuinely independent code path. This is IBM support-case territory if the built-in namespace specifically needs to work. |
| 7 | `DPR-ERR-2002` — reports fail to run, generic error | Report-rendering engine (`BIBusTKServerMain`) is a 32-bit binary; RHEL 9.2 was 64-bit-only with none of the required compatibility libraries | Install matching `.i686`/`.x86_64` glibc, libstdc++, ncurses-libs, and libnsl packages together (Section 10.1) |

> 📌 **NOTE on Issue 6**
> Before concluding this needs LDAP, the following were all checked and ruled out with direct evidence: system clock (NTP-synced, correct), database connectivity (schema builds cleanly every time), schema permissions (fixed, confirmed via `psql`), CAF domain allow-list (populated correctly), Environment URIs (corrected to IP), and IBM's own `dbClean_postgresql.sql` script (run per official guidance, no change). If you hit this exact symptom, don't assume it's fixable at the configuration level — the evidence here says it likely isn't, at least not without vendor support.

> 📌 **NOTE on Issue 7**
> This is a documented OS prerequisite gap, not a corrupted download or installer malfunction — the installer copies every file correctly. Confirmed: the Cognos installer media was intact, `BIBusTKServerMain` was present with correct ownership, and the fix was purely about OS-level 32-bit compatibility libraries. Worth adding to any future pre-installation checklist rather than discovering it reactively during report testing.

---

## 12. Appendix — Quick Reference

| Item | Value |
|---|---|
| Install location | `/opt/ibm/cognos/analytics` |
| Config tool | `bin64/cogconfig.sh` |
| Web UI | `http://<ip>:9300/bi` |
| Content Manager URI | `http://<ip>:9300/p2pd/servlet` |
| Content store | PostgreSQL 16, database `cognoscs`, schema `public` |
| Auth | LDAP (`LDAPTest` namespace), base DN `dc=fpclab,dc=local` |
| Package source | Nexus, `192.168.2.90:8081` |
| Logs | `install_location/logs/cognosserver.log` |
| JDBC driver | `postgresql-42.7.13.jar` (Java 8 / JDBC 4.2 — compatible with the bundled Semeru 17 JRE) |

---

*Screenshots marked as placeholders above (if any remain unresolved paths) should be dropped into `./images/` with the exact filename referenced in the markdown — they'll render automatically once present.*
