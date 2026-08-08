# Concord Energy — ClearSky-OMEGA Portal

Client deployment of the ClearSky-OMEGA EnergyOS portal for **Concord Energy** —
a new company focused on sales and energy projects.

Four unlocked tools on a 14-day trial, with a one-line switch to convert to a
paid Tier 1 account.

Mail domain: **`concordenergyusa.com`**. Note the `usa` — `concordenergy.com`
is a live domain belonging to an unrelated Denver oil-and-gas trading firm, so
don't shorten it. Both `orgId` and `allowedDomain` are set to the full form and
must stay identical.

Branding is Concord's own concentric-C mark (`concord-logo.png`), used for both
the portal chrome and proposal exports. No placeholders remain — this repo is
ready to deploy.

---

## Converting to Tier 1

One line, at the top of `config.js`:

```js
var PLAN = 'trial';   // ← change to 'tier1'
```

Commit, redeploy, done. No shared file is touched, no other line in `config.js`
needs editing, and it takes effect on the next page load.

| | `'trial'` | `'tier1'` |
|---|---|---|
| `accountTier` | Trial | Standard |
| `tierLevel` | -1 | 1 |
| Countdown banner | yes | none |
| Tools unlocked | **4** | **16** |
| Tools Upgrade-badged | 29 | 17 |

Both paths verified against the live registry at `tools.csebuilders.com/omega-tools.js`.
A misspelled `PLAN` logs a named console error and falls back to `trial` rather
than silently producing a tenant with no tier and no tools.

### What Tier 1 actually buys them

This is the commercial substance of the upgrade — twelve tools on top of the
trial four:

| Key | Tool |
|---|---|
| `gridatlas` | Grid Atlas |
| `sandbox` | Open a Sandbox |
| `proforma` | BESS Pro Forma |
| `valuestack` | Value Stack Calculator |
| `isocalc` | BESS ISO Calculator |
| `datacenter` | Data Center Compute Calculator |
| `signal` | OMEGA Signal |
| `interconnect` | Interconnection Screener |
| `ahj` | AHJ Approval Portal — *renders "Soon", not yet clickable* |
| `procurement` | Procurement Marketplace — *"Soon"* |
| `aggregators` | Aggregators — *"Soon"* |
| `offtakers` | AI Data Offtakers — *"Soon"* |

Worth knowing before you price it: four of those twelve are "Soon" placeholders
that no tenant can open yet. The live gain is **eight** working tools, not twelve.

Tier 2 (`tierLevel: 2`) would take it to 31, Tier 3 to all 33. Neither has a
preset here — add one to the `PLANS` map in `config.js` if you need it.

---

## Trial

| | |
|---|---|
| Account tier | **Trial** (`tierLevel: -1`) |
| Starts | **Mon Aug 3, 2026** |
| Length | **14 days** |
| Last full day | **Sun Aug 16, 2026** |
| Expires | **Mon Aug 17, 2026, 00:00** local |
| On expiry | Banner only — access continues (`lockOnExpiry: false`) |

Aug 3 is carried over from the FENECON deployment since you said "same trial" —
**change `startsAt` in `config.js` if Concord's start differs.** A new company
may need longer to get people onboarded before the clock is worth starting.

Banner states: blue before Aug 3, amber Aug 3–9, red Aug 10–16, grey from Aug 17.

**One quirk of a 14-day trial:** the red "ending soon" state is hardcoded at
7 days or fewer in `omega-brand.js`, which is a shared file and can't be tuned
per tenant. On a 30-day trial that's the final quarter; on this one it's the
back half. Concord will see an urgent red banner for half their trial. If that
reads as too pushy, the options are a longer trial or a change upstream in
`omega-brand.js` that applies to every tenant.

To harden expiry into an actual lockout, set `lockOnExpiry: true` inside the
`trial` block. Domains in `adminDomains` keep access either way.

---

## What's in here

| File | Shared? | Notes |
|---|---|---|
| `index.html` | **shared** | Portal dashboard |
| `marketplace.html` | **shared** | App marketplace |
| `projects.html` | **shared** | Project list |
| `editor.html` | **shared** | BESS Site Map application |
| `omega-brand.js` | **shared** | Tenant resolution + branding |
| `config.js` | **tenant-specific** | The only file to edit |
| `concord-logo.png` | tenant asset | Concord's concentric-C mark |
| `omega-logo.png` | platform asset | ClearSky-OMEGA mark |

The five shared files are byte-identical to the FENECON and iQGen deployments —
verified by checksum before this repo was cut. Fixes belong upstream and get
copied down; never patch them here, or this repo silently forks.

**About the logo file.** Built from the screenshot you supplied: white
background keyed to transparent, trimmed to the mark, 5% margin, output at
218x264.

The source mark measured only 135x167px, so the file is already a 1.5x upscale.
It's sized at 264px tall to cover the 88px sign-in card at 3x DPR, and pushing
it larger would add blur rather than detail. If it reads soft on a retina
screen, ask Concord for the vector original (SVG/AI/EPS) and re-export — that's
the only real fix.

One thing to know before anyone re-processes this file: the light grey in the
outer arc is `#BEC4C3`, only about 106 units from white in RGB. A naive
white-removal pass would eat most of that arc. The threshold used here was
deliberately tight (keys out only pixels within ~8 units of pure white, with a
feathered ramp to 26) to preserve it alongside `#6F7D7A` slate and `#E5AB4B`
amber.

The mark is a symbol with no wordmark, so `clientName` ("Concord Energy") does
the naming work in the sidebar and on the sign-in card.

---

## Before this goes live

1. **DNS** — add a CNAME `concord` in the `clearskyomega.com` GoDaddy zone
   pointing at whatever target Vercel issues for this project. The hash is
   per-domain, so copy it from Vercel rather than from another tenant's record.
2. **Firebase authorized domains** — Console → Authentication → Settings. Add
   both `concord.clearskyomega.com` **and** `concord.vercel.app`. Missing the
   raw Vercel URL is the failure mode where the page renders fine and Google
   sign-in errors out.
3. **Firestore rules** — confirm `userOrg()` maps `@concordenergyusa.com` to the
   `concordenergyusa.com` org.
4. **Seed their projects** with `orgId: 'concordenergyusa.com'`, or the
   portal authenticates fine and shows an empty portfolio. A brand-new company
   may have nothing to import — in which case check that an empty portfolio
   still reads as a clean starting state rather than a loading failure.
5. **Run "Import / Update Applications"** in the admin console if the live
   marketplace shows fewer than 33 tools — the portal hydrates its catalog from
   the Firestore `tools` collection whenever that's non-empty, and Firestore has
   historically lagged the seed in `omega-tools.js`.

If DNS gives you `DNS_PROBE_FINISHED_NXDOMAIN` right after you add the record,
that's a cached negative response on your machine, not a zone problem — flush
with `sudo dscacheutil -flushcache; sudo killall -HUP mDNSResponder`, then turn
off Chrome's secure DNS at `chrome://settings/security`, which keeps a separate
cache the system flush doesn't touch.

---

## Access

Sign-in is restricted to `@concordenergyusa.com`. If they add a second domain
later, uncomment `allowedDomains` and list the extras — they all land in the
same workspace, since `orgId` is fixed regardless of which address signs in.

`csebuilders.com` and `clearsky-usa.com` may preview and survive expiry.

To admit an individual outside address — likely for a new company still standing
up its mail — add it to the tenant rather than opening a whole domain:

```js
allowedEmails: ['someone@gmail.com']
```

---

## Tools unlocked during the trial

| Key | Tool | Category |
|---|---|---|
| `editor` | BESS Site Map | design — also pinned via `requiredTools` |
| `batterysizer` | Battery Sizer | finance |
| `sales` | Sales Proposal Builder | sales |
| `financing` | Financing Partners | marketplace |

Same four as FENECON and iQGen. `spatco_ev` stays hidden — it's `orgs`-restricted to
another tenant and never appears here.

Two of the keys aren't literal matches for how these tools get described in
conversation, and are worth knowing about before you edit the list:
**`batterysizer`** is the "BESS sizer" (not `isocalc` or `proforma`, both of
which also carry "BESS" in their names), and **`financing`** is the "financial
marketplace" (the other marketplace entries all render "Soon").

### The gate

From `omega-tools.js`:

```
unlocked = requiredTools.has(key)
        || unlockedTools.has(key)
        || tierLevel >= (tool.tier ?? 1)
```

Tiers are `ALL=0`, `STANDARD=1`, `DELUXE=2`, `ENTERPRISE=3`. That third clause is
the whole mechanism behind the plan switch: at `-1` nothing passes on tier and
only the explicit list counts; at `1` every tier-0 and tier-1 tool opens.

`unlockedTools` stays populated under `tier1` even though tier alone would cover
all four. It's redundant but deliberate — it means those four survive any future
retiering of a tool upstream in `omega-tools.js`.

---

## Terms of Service gate

New accounts must accept Terms of Service before the portal renders. Two shared
files carry this — both byte-identical across every tenant:

| File | Role |
|---|---|
| `omega-terms.js` | The gate: consent checkbox, terms modal, Firestore record |
| `firestore-terms.rules` | The rule that must be deployed for it to work |

`index.html` gained exactly one `<script>` tag; nothing else in it changed.

**Two layers, deliberately.** The sign-up form gets a consent checkbox that
blocks account creation while unticked. But the real enforcement is a gate that
runs after authentication and before the app renders — because a checkbox on the
sign-up form would miss Google sign-in entirely (a first-time Google user never
sees that form) and would miss version bumps.

**Acceptance is recorded**, which is the part that gives it weight: uid, email,
orgId, version and a server timestamp land at `termsAcceptances/{uid}`. A
checkbox nobody stored is close to worthless in a dispute.

**Amending the terms:** bump `TERMS_VERSION` at the top of `omega-terms.js`.
Every user is re-prompted on their next load. The rule permits an update only
when the version string actually changes, so an existing acceptance can't be
silently rewritten with a fresh timestamp.

### ⚠ Deploy the rule

```
firebase deploy --only firestore:rules
```

Until `termsAcceptances` is live in Firebase the acceptance write returns
permission-denied and the gate **fails closed — nobody can sign in, on any
tenant**. That direction is deliberate (failing open would let people through
ungated), but it means a forgotten rules deploy looks like a total outage. The
modal names the missing rule when it happens. Confirm the rule appears in
Firebase Console → Firestore → Rules before calling it done.

### Not legal advice

The terms are a standard SaaS starting point covering platform IP ownership,
licence scope, use restrictions (no reverse engineering, resale, white-labelling
or competing use), customer data ownership, confidentiality, trial terms, and an
engineering-output disclaimer stating that generated site plans, one-lines and
pro formas are estimates rather than sealed engineering documents. **Have a
lawyer review before relying on any of it.** Two placeholders are marked REVIEW
in the file: governing law and venue (currently Iowa) and the formal notice
address (currently `dev@clearsky-usa.com`).
