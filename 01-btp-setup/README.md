# 01 — SAP BTP Trial Setup

Setting up a free SAP BTP trial with SAP Integration Suite, from nothing to a
tenant you can build integration flows in.

Written as a runbook rather than a walkthrough, because a BTP trial expires after
90 days and this has to be done again. The **Gotchas** section at the bottom is
the part worth reading — the official documentation covers the happy path, and
none of the three things that actually cost me time are in it.

| | |
|---|---|
| Set up on | 16 September 2026 |
| Region | `<fill in: e.g. US East (VA) – AWS>` |
| Expires | ~15 December 2026 (90 days) |
| Cost | Free |

---

## What this produces

- A BTP trial global account, subaccount and Cloud Foundry space
- SAP Integration Suite subscribed and its capabilities activated
- Role collections assigned so the tooling is actually usable
- One working destination, connection-tested
- A tenant where integration flows can be designed, deployed and monitored

---

## Steps

### 1. Activate the trial

Sign up at `cockpit.hanatrial.ondemand.com` and pick a region. The trial gives a
global account with one subaccount.

The 90-day clock starts the moment the trial is activated, so it is worth
activating only when you are ready to use it.

![Account hierarchy](img/01-account-hierarchy.png)

### 2. Understand the hierarchy before touching anything

```
Global account          billing and entitlements live here
  └─ Subaccount         region-specific; services are subscribed here
      └─ CF space       runtime; apps and instances live here
```

Most "why can't I do this" problems later turn out to be a question of which
level you are standing on.

### 3. Check entitlements

**Entitlements → Service Assignments.**

Two words that are easy to confuse and worth separating early:

- **Entitlement** — permission to use a service
- **Quota** — how much of it you may use

A service with an entitlement but no quota behaves exactly like a broken service.

For the trial the relevant line is:

```
Integration Suite   plan: trial   1 unit   (0 used)
```

![Entitlements](img/02-entitlements.png)

### 4. Create a Cloud Foundry space

Create a space (`dev`) and assign yourself **Space Developer**.

### 5. Subscribe to Integration Suite

**Service Marketplace → Integration Suite → Create**, plan `trial`.

Wait for the subscription status to reach **Subscribed** before continuing.

![Subscriptions](img/03-subscriptions.png)

### 6. Assign `Integration_Provisioner` — before anything else

**Security → Role Collections → `Integration_Provisioner`** → assign to your user
→ **log out and log back in**.

This step is not optional and it is not obvious. See Gotchas.

![Role collections](img/04-role-collections.png)

### 7. Activate capabilities

Open Integration Suite and activate:

| Capability | Why |
|---|---|
| Build Integration Scenarios | Cloud Integration — integration flows. The core. |
| Manage APIs | API Management — proxies, policies, Developer Hub |
| Implement Interfaces and Mappings | Integration Advisor — MIGs and MAGs, needed for EDI |
| Manage Trading Partners | Trading Partner Management — B2B / EDI scenarios |
| Extend Non-SAP Connectivity | Open Connectors — third-party systems |

Prerequisite capabilities are co-activated automatically, so the final list may be
longer than what you ticked.

![Capabilities](img/05-capabilities.png)

### 8. Assign the capability role collections

Activating a capability creates new role collections (`PI_Integration_Developer`,
`PI_Administrator`, the API Management ones, and so on). Assign them to your user,
then **log out and log back in again**.

### 9. Create a destination

**Connectivity → Destinations → Create**, pointed at any public REST API, then
**Check Connection** until it returns success.

Destinations are how integration flows reach the outside world, so it is worth
having one working before you need one.

![Destination](img/06-destination.png)

### 10. Verify

**Design → Integrations** should open and let you create a package. If it does,
the tenant is ready.

![Integration Suite](img/07-integration-suite.png)

---

## Gotchas

**Subscribed does not mean usable.** The subscription reaching *Subscribed* is not
the finish line. Without the `Integration_Provisioner` role collection you can open
Integration Suite but cannot activate any capability, and the failure does not
explain itself. Assign that role collection first.

**Every role assignment needs a fresh login.** Role collections are read into the
user session at login. Assign a role, keep working in the same session, and it
looks exactly as though the assignment did nothing.

**SAP Business Application Studio failed to subscribe, and it did not matter.**
BAS subscription failed on this trial — capacity, most likely. It is not needed for
Integration Suite work: integration flows are built in Integration Suite's own web
designer, and ABAP development uses Eclipse with ADT. Worth knowing before losing an
afternoon to it.

---

## Trial lifecycle

The trial runs 90 days and can be extended, but a trial subaccount can also be
removed after a period of inactivity.

**Nothing built here is safe.** Every integration flow gets exported and committed
to this repository as it is built, not at the end — anything that exists only inside
the trial is temporary by definition.

---

## Next

`02-first-iflow/` — HTTP to HTTP passthrough, deployed, called from Postman, and
traced through the Message Processing Log.
