# SaaSassin Revenue

**Free, open-source Gong alternative.** Turn customer conversations into inspectable revenue coaching on infrastructure you control.

Revenue intelligence for B2B sales teams that need call intelligence without opaque scoring or uncontrolled recordings.

- **Try it:** [revenue.saasassin.net](https://revenue.saasassin.net)
- **Research brief and product decision:** [saasassin.net/targets/gong](https://saasassin.net/targets/gong)

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/new/clone?repository-url=https%3A%2F%2Fgithub.com%2Falibad%2Fsaasassin-revenue&project-name=saasassin-revenue&repository-name=saasassin-revenue&env=NEXT_PUBLIC_FIREBASE_API_KEY%2CNEXT_PUBLIC_FIREBASE_AUTH_DOMAIN%2CNEXT_PUBLIC_FIREBASE_PROJECT_ID%2CNEXT_PUBLIC_FIREBASE_APP_ID%2CFIREBASE_PROJECT_ID%2CFIREBASE_CLIENT_EMAIL%2CFIREBASE_PRIVATE_KEY&envDescription=Firebase+web+app+config+and+service-account+credentials+%28see+the+README%29&envLink=https%3A%2F%2Fgithub.com%2Falibad%2Fsaasassin-revenue%231-create-a-firebase-project)

Gong is a trademark of its owner and is named here only to describe what this product
replaces. SaaSassin Revenue is an independent, clean-room product and is not affiliated
with or endorsed by Gong.

## What you can do today

### What users can do now (round 1)

- **Sign in and create a team workspace**; switch between isolated workspaces; sign out.
- **Build the pipeline**: add accounts, then deals with stage, amount, close date and owner.
  The deal board shows every stage column with count and value, each deal's last call, and
  **risk signals** (no calls, no call in N days, no next step in the latest call, competitor
  mentioned in the latest call, close date passed). Each signal says which rule fired and
  links to the call (and timestamp) that caused it.
- **Move deals through stages** with a server-enforced stage machine and a full stage history
  (who, when, from → to). Closed deals can only be reopened by an owner/admin.
- **Log a call against a deal** by pasting or uploading a transcript (WebVTT, SRT, or
  `Speaker: text` lines with optional `[mm:ss]` stamps). No audio is processed and no bots join
  meetings. **A recording-consent record is mandatory**: jurisdiction, consent basis (all-party
  verbal / written / notice-and-continue) and an all-party confirmation. The server rejects
  transcripts without it (422); the UI keeps the submit disabled until it is complete.
  The form previews the parse and analysis live in the browser before saving.
- **Automatic deterministic analysis** (pure module, same input → same output): talk ratio per
  speaker, longest monologue, question count, interactivity (speaker switches/min scaled to
  0–10), keyword/competitor tracker hits with timestamps and character offsets, and next steps
  detected by named phrase rules.
- **Review a call**: transcript with tracker highlights (click a tracker to focus it), next-step
  badges, clickable timestamps that jump to the evidence, consent record, and stats.
  Reps add **comments at a timestamp** (or on the whole call).
- **Scorecards**: managers/admins score calls against a weighted 1–5 rubric; the percent and
  notes show on the call and roll into the dashboard.
- **PII redaction**: managers/admins redact an exact span of a segment. The text is replaced by
  `[REDACTED]`, never retained, analysis is recomputed, and the audit log records who, where,
  length and reason — not the removed text.
- **Search every transcript** for a word or phrase and open the call scrolled to the match.
- **Coaching clips & playlists**: reps clip a range of segments into a named playlist with a
  note; the Coaching tab groups clips by playlist and opens each at its moment. Clips store
  timestamps only, so redactions and deletions always apply.
- **Coaching assignments**: managers assign a call (optionally at a timestamp) to a rep with a
  focus note; the rep acknowledges it from their queue. Only the assignee can acknowledge.
- **Editable scorecard rubric** (owners/admins): criteria and weights; saved scorecards keep the
  rubric they were scored with.
- **Retention & deletion**: owners/admins set a retention period, preview exactly which calls
  it would delete, and apply it; single calls can be deleted with a recorded reason. Deletes
  cascade to comments, scorecards, clips and assignments; the audit keeps metadata only.
- **CRM write preview**: each deal lists the CRM fields an approved sync would write (stage,
  amount, close date, last call date, next step, competitors) with a link to the transcript
  moment behind each call-derived value. No CRM is connected, so nothing is written — the
  Integrations tab says so plainly, as it does for Vexa meeting capture.
- **Configurable trackers** (owners/admins): create/edit/delete keyword or competitor trackers;
  re-analyze any call with the current trackers. Stale-call threshold is configurable.
- **Team dashboard** computed only from this workspace's logged calls: per-rep calls, minutes,
  average rep talk ratio, longest monologue, questions/call, interactivity, next-step rate and
  average score — with explicit metric and risk-rule definitions on the page.
- **Members & roles**: owner/admin grant access by email as admin, manager, rep or viewer.
- **Audit log** view (owner/admin/manager) of every mutation; transcript text is never written
  to the audit log.
- **Export**: owners/admins download a complete versioned JSON export
  (`net.saasassin.workspace-export` manifest + accounts, deals, calls with transcripts and
  consent, comments, scorecards, trackers, settings, members, audit events).

### Round 2 — what changed for users

- **Proven on a real Firestore.** `tests/revenue.emulator.spec.ts` drives the real data layer
  (27 tests) through the whole golden workflow and every round 2 slice, with role denials for
  admin/manager/rep/viewer, outsider 404s, cross-tenant isolation, audit events and the export.
- **Fixed: concurrent redactions could restore removed PII.** Two coaches redacting the same call
  at once each wrote back their own stale copy, so the later write put the first redacted text
  (a phone number in the test) back into storage. Redaction now runs in a transaction and carries
  a fingerprint of the segment text it was selected from; a stale selection is refused (409)
  instead of being applied to different text. Stage moves, coaching acknowledgement and
  first-load seeding are now transactional too (no lost stage history, trackers seeded once).
- **Role checks live in the data layer** as well as the routes, so the spec exercises the same gates.
- **Contacts**: add people to accounts (name, title, optional email) and give each a role per deal
  (champion, decision maker, economic buyer, influencer, blocker, end user). On a call, link each
  transcript speaker to a contact on the deal's account or to a teammate; exact-name matches are
  offered as one-click suggestions but never applied automatically. Contacts show talk time,
  calls, share of talk and last call. Deleting a contact unlinks it from every call.
- **Multi-threading risk**: an open deal whose calls have heard exactly one linked customer contact
  gets a `single_threaded` signal pointing at that contact's latest call. Unlinked speakers are
  never counted, so the rule only fires on evidence.
- **Forecast tab**: commit, best case, pipeline, weighted pipeline and closed won — overall, by
  close month and by rep — computed only from real deals. Weighted = amount × the workspace's
  stage probability (owners/admins edit them; won 100%, lost 0%). The deal owner calls the
  category; managers/admins/owners override it with a required reason. The owner's call stays
  visible, and calls, overrides, clears and probability edits are all audited.
- **Deal timeline** in the deal dialog: stage changes, calls, comments and detected next steps in
  one stream (each jumps to the call moment), time-in-stage bars per stage (with revisits), deal
  age, and a stalled badge. A `stalled_in_stage` risk fires past a configurable threshold
  (default 30 days); cards show days in stage and contact count.
- **Signed webhooks** (Integrations, owners/admins): `call.analyzed` (on log, re-analyze and
  redaction — analysis numbers only, never transcript text or next-step quotes) and
  `deal.stage_changed`. `X-SaaSassin-Signature: t=..,v1=<HMAC-SHA256(secret, "t.body")>`.
  HTTPS on 443 only; private/internal hostnames and literal private IPs are refused when saving,
  and the host is resolved again at delivery so DNS pointing inside is refused. The secret is
  shown once; reads show a hint and the export says `[redacted]`. Pause, send test, delete, and a
  delivery log (latest 50) that records delivered/failed with the real reason.
- **Retention made visible**: admins see a banner when calls are past retention and a queue on
  Trackers & settings listing each due call, its due date and days overdue, plus calls due in the
  next 14 days. Delete one call or all due calls (confirmed); the server refuses calls that are
  not actually due. Still admin-triggered, never automatic; audited with scope and call ids.
- **Export** now also carries `contacts`, `webhooks` (secret `[redacted]`) and
  `webhookDeliveries`; deals include forecast calls/overrides and calls include speaker links.

Full status, verification and known gaps: [docs/STATUS.md](docs/STATUS.md).

## Self-host

SaaSassin Revenue is a Next.js app backed by Firebase Authentication and Cloud Firestore.
You need Node.js 20.9 or newer, [pnpm](https://pnpm.io/installation) and a Firebase project of
your own.

### 1. Create a Firebase project

1. Create a project in the [Firebase console](https://console.firebase.google.com/).
2. **Firestore:** Build → Firestore Database → Create database.
3. **Authentication:** Build → Authentication → Get started. Revenue signs people in with
   **Google** and **Email/Password**; step 2 below enables both, or enable them yourself under
   Authentication → Sign-in method.
4. **Web app config:** Project settings → General → Your apps → add a Web app. Copy its config
   into the `NEXT_PUBLIC_FIREBASE_*` variables.
5. **Service account:** Project settings → Service accounts → Generate new private key. From the
   downloaded JSON, set `FIREBASE_PROJECT_ID` (`project_id`), `FIREBASE_CLIENT_EMAIL`
   (`client_email`) and `FIREBASE_PRIVATE_KEY` (`private_key`). Never commit the key, and never
   give these variables a `NEXT_PUBLIC_` prefix.
6. Once you deploy, add your domain under Authentication → Settings → Authorized domains
   (`localhost` is already allowed).

### 2. Deploy the rules, indexes and sign-in providers

```bash
pnpm install
pnpm exec firebase login
pnpm exec firebase deploy --only firestore:rules,firestore:indexes,auth --project <your-project-id>
```

`firestore.rules` denies all browser access to product data: every read and write goes through the server routes, which check the session, tenant membership and role with the Firebase Admin SDK. `auth` enables the Google and email/password providers listed in `firebase.json`.

### 3. Run it

```bash
cp .env.example .env.local   # fill in the values from step 1
pnpm dev
```

Open [http://localhost:3000](http://localhost:3000). This repository serves SaaSassin
Revenue at the root of every host: `next.config.ts` sets `STANDALONE_PRODUCT=revenue`,
and `src/proxy.ts` maps `/` to the product (`/apps/revenue`) and `/sign-in` to its sign-in page.

### 4. Deploy to Vercel

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/new/clone?repository-url=https%3A%2F%2Fgithub.com%2Falibad%2Fsaasassin-revenue&project-name=saasassin-revenue&repository-name=saasassin-revenue&env=NEXT_PUBLIC_FIREBASE_API_KEY%2CNEXT_PUBLIC_FIREBASE_AUTH_DOMAIN%2CNEXT_PUBLIC_FIREBASE_PROJECT_ID%2CNEXT_PUBLIC_FIREBASE_APP_ID%2CFIREBASE_PROJECT_ID%2CFIREBASE_CLIENT_EMAIL%2CFIREBASE_PRIVATE_KEY&envDescription=Firebase+web+app+config+and+service-account+credentials+%28see+the+README%29&envLink=https%3A%2F%2Fgithub.com%2Falibad%2Fsaasassin-revenue%231-create-a-firebase-project)

The button copies this repository into your GitHub account, creates a Vercel project and asks for
the Firebase variables. To deploy an existing fork instead, import it in Vercel and add the
variables from `.env.example` under Project Settings → Environment Variables. Either way, add the
Vercel domain to your Firebase authorized domains afterwards.

## Tests

```bash
pnpm lint
pnpm typecheck
pnpm test        # unit tests
pnpm build
```

The Firestore suites run against the local Firestore emulator, which needs a Java 21+ JDK first on
your `PATH`:

```bash
export JAVA_HOME=/path/to/jdk-21   # macOS example: /Library/Java/JavaVirtualMachines/temurin-21.jdk/Contents/Home
export PATH="$JAVA_HOME/bin:$PATH"
pnpm test:rules      # Firestore security rules
pnpm test:emulator   # the golden workflow against a real Firestore emulator
```

## Contributing

Development happens in the SaaSassin monorepo,
[github.com/alibad/saasassin](https://github.com/alibad/saasassin), the single source of truth for
all ten SaaSassin products. This repository is generated from it by the monorepo's
`scripts/export-product.mjs`, so accepted changes land in the monorepo first and are then synced
here. Open issues and pull requests there or here.

Report vulnerabilities privately as described in [SECURITY.md](SECURITY.md).

## Talk to us

Moving off Gong, running Revenue for your team, or want to help build it?
[Tell us on saasassin.net](https://saasassin.net/targets/gong#interest).

## License

[GNU Affero General Public License v3.0](LICENSE) (AGPL-3.0-only). Use it, self-host it and modify
it; if you run a modified version as a network service, publish your changes under the same
license.
