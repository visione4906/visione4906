# Sal Oladapo

I build LLM systems that run in production, and I keep a person at the one gate where
a mistake would be public.

Python, FastAPI, Anthropic Claude, TypeScript, Next.js, SQLite, Oracle Cloud ARM,
Cloudflare. Based in the UK.

**[demo.consentleads.uk](https://demo.consentleads.uk) is live.** Pick a sector,
message it the way a customer would, and read what comes back. Nothing is scripted.

---

### What is here

- **[A live service that answers for small businesses](#a-live-service-that-answers-for-small-businesses).** Running unattended on nine scheduled jobs, with one paying client.
- **[claimcheck](#claimcheck).** Verifies a document against the code it describes, and fails the build when it lies.
- **[Testing the failures that stay quiet](#testing-the-failures-that-stay-quiet).** How the service above avoids breaking without anyone noticing.
- **[A video pipeline, and the fork that proved it](#a-video-pipeline-and-the-fork-that-proved-it).** Script to upload, on a schedule.
- **[Skin](#skin).** Money behind a promise, enforced by a contract. Solidity and TypeScript.

---

## A live service that answers for small businesses

Live at [demo.consentleads.uk](https://demo.consentleads.uk). The code is private; it
runs the business.

A service business that misses an enquiry usually loses it. The customer books
whoever answers first, and out of hours that is nobody. This answers straight away,
in the business's own voice, and carries the conversation through to a booking and a
payment.

It runs on FastAPI and Anthropic Claude. A single prompt builder serves every customer
from their own configuration, switching between a concierge path and a booking path.
Behind that sit twenty-four service modules and a provider layer, so a vendor can be
replaced without rewriting the app: Stripe for payments, with an end-to-end smoke test
and a refund tester; Resend and Twilio for email and SMS, including missed-call
recovery; rolling and static calendar slots; Instagram OAuth, sending and inbound
webhooks; and the account layer, covering authenticated dashboards, onboarding, rate
limiting and suppression.

Consent lives in the code rather than in a policy document. The suppression gate fails
closed, so a contact who has opted out cannot be messaged even when something upstream
is broken. That property is what would survive a regulated review.

It is built multi-tenant and currently carries one paying client. It sits on a
free-tier ARM box behind a Cloudflare tunnel, and the health check answered in 0.16
seconds on 2 August 2026. Nine scheduled jobs keep it running without supervision:
reply triage six times a day, bounce handling, deliverability reporting, nurture
sequences, weekday sourcing, a nightly backup, a monthly unit-economics run, and a
watchdog every fifteen minutes.

## claimcheck

[github.com/visione4906/ctaio-claimcheck](https://github.com/visione4906/ctaio-claimcheck).
Public, MIT licensed.

An agent writes the code, then the README, then the release notes, and somewhere along
the way it starts describing the system it meant to build rather than the one that
exists. Nobody reads it back against the source. The claim survives into the
changelog, the pitch, and eventually the CV.

claimcheck takes a document and a codebase, and makes a second, independent agent
prove every claim against the source with file and line citations, or refuse it. It
exits with an error on anything unsupported, so it can sit in a build pipeline and
stop a release.

I built it after an outside review of my own writing turned up claims the code did not
support. Being more careful is a promise. This is a check that runs every time.

## Testing the failures that stay quiet

Same private codebase as the service above.

The failures that matter in an LLM system are the ones that do not announce
themselves. If opt-out detection misses a phrase, the system carries on emailing
somebody who asked it to stop. Nothing crashes, no alert fires, and the cost arrives
weeks later on the sending domain.

So the evaluation fixtures cover exactly two paths, the two where a regression is both
silent and expensive. Each case is a single line of JSON pinning behaviour the code
already has, which turns a regression into a failing check rather than a live
incident.

- `suppression_classify.jsonl`, recognising an opt-out in an inbound reply
- `reply_triage_classify.jsonl`, sorting positive, opt-out, bounce and out-of-office

A test suite covers the same class of risk: the suppression gate, rate limiting,
dashboard auth tokens, booking slot logic, a browser security header the live demo
depends on, and what the service does when a model call fails outright.

Two paths, chosen deliberately over a coverage number. These are the ones that fail
without telling you.

## A video pipeline, and the fork that proved it

[github.com/visione4906/horror-shorts-pipeline](https://github.com/visione4906/horror-shorts-pipeline).
Public, MIT licensed.

A channel needs output on a schedule, and by hand that costs hours per video. This
runs the whole chain: seed title, written script, narration, word-level timing,
generated images, assembly, then a scheduled upload. Progress is stored at each step,
so a failure resumes where it stopped instead of starting over.

One step stays manual. Every video comes to me on Telegram for approval before it
publishes. A single bad title is a strike against the channel, and that is not a risk
worth automating away.

Six scheduled tasks run it, all confirmed active on 2 August 2026, and it has
published real videos to a real channel. I later forked it for two brands in an
unrelated subject, reusing the same staged steps, which is the evidence that it
generalises. I would rather show that than assert it.

## Skin

[github.com/visione4906/skin](https://github.com/visione4906/skin). Public.

Deciding to do something is easy. Following through is the part that fails, and a
promise to yourself costs nothing to break. Skin puts money behind the promise. Keep
it and you take the money back. Miss your own deadline and anyone can send the stake
to the person you named, keeping five percent for doing it. The consequence is
automatic, so nothing rests on willpower after the fact.

A Solidity contract with a Foundry test suite, and a Next.js front end in TypeScript
using wagmi and viem for the wallet and contract calls. Designed and shipped inside a
hackathon week for BuildAnything Spark. The pledges are testnet MON, not real money.

It also taught me something I have kept: test through the path a user actually takes.
Calling the contract directly passed cleanly. Only clicking through the wallet showed
what that call could not.

---

## Contact

[LinkedIn](https://www.linkedin.com/in/oladapo678). Open to AI engineering and AI
automation roles, remote.

More public repositories are on
[my profile](https://github.com/visione4906?tab=repositories).
