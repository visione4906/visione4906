# Sal Oladapo

I build LLM systems that run in production, and I keep a person in the loop at the one
point where a mistake would be public.

Python, FastAPI, Anthropic Claude, TypeScript, Next.js, SQLite, Oracle Cloud ARM,
Cloudflare. Based in the UK.

**[demo.consentleads.uk](https://demo.consentleads.uk) is live.** Pick a sector,
message it the way a customer would, and read what comes back. Nothing is scripted.

---

### What is here

- **[A live service that answers for small businesses](#a-live-service-that-answers-for-small-businesses).** Running unattended on scheduled jobs.
- **[claimcheck](#claimcheck).** Verifies a document against the code it describes, and fails the build when it lies.
- **[Testing the failures that stay quiet](#testing-the-failures-that-stay-quiet).** How the service above avoids breaking without anyone noticing.
- **[A video pipeline, and the fork that proved it](#a-video-pipeline-and-the-fork-that-proved-it).** Script to upload, on a schedule.
- **[Skin](#skin).** Money behind a promise, enforced by a contract. Solidity and TypeScript.
- **[brain-engine](#brain-engine).** A decision engine in Rust, and the MCP server that exposes it as eight tools over stdio.

---

## A live service that answers for small businesses

Live at [demo.consentleads.uk](https://demo.consentleads.uk). The code is private; it
runs the business.

A service business that misses an enquiry usually loses it. The customer books
whoever answers first, and out of hours that is nobody. This answers straight away,
in the business's own voice, and carries the conversation to the point of a booking.

It runs on FastAPI and Anthropic Claude. A single prompt builder serves every customer
from their own configuration, switching between a concierge path and a booking path.
Behind that sit twenty-three service modules and a provider layer, so a vendor can be
replaced without rewriting the app: Stripe for payments, with an end-to-end smoke test
and a refund tester; Resend and Twilio for email and SMS, including missed-call
recovery; rolling and static calendar slots; Instagram OAuth, sending and inbound
webhooks; and the account layer, covering authenticated dashboards, onboarding, rate
limiting and suppression.

Consent lives in the code rather than in a policy document. Every outbound message has
to pass a suppression gate, and a genuine human reply puts that person on a persistent
list that every later send checks.

It is built multi-tenant. It sits on a
free-tier ARM box behind a Cloudflare tunnel, and the health check answered in 0.16
seconds on 2 August 2026. Scheduled jobs keep it running without supervision: reply
triage six times a day, bounce handling, deliverability reporting, a nurture sequence,
a monthly unit-economics run, and a watchdog every fifteen minutes.

## claimcheck

[github.com/visione4906/ctaio-claimcheck](https://github.com/visione4906/ctaio-claimcheck).
Public, MIT licensed.

An agent writes the code, then the README, then the release notes, and somewhere along
the way it starts describing the system it meant to build rather than the one that
exists. Nobody reads it back against the source. The claim survives into the
changelog, the pitch, and eventually the CV.

claimcheck takes a document and a codebase, and makes a second, independent agent
prove every claim against the source with file and line citations, or refuse it. Each
claim comes back true, an overclaim, or unverifiable, and it exits with an error on
anything the evidence contradicts, so it can sit in a build pipeline and stop a
release.

I built it after an outside review of my own writing turned up claims the code did not
support. Being more careful is a promise. This is a check that runs every time.

## Testing the failures that stay quiet

Same private codebase as the service above.

The failures that matter in an LLM system are the ones that do not announce
themselves. If opt-out detection misses a phrase, the system carries on emailing
somebody who asked it to stop. Nothing crashes, no alert fires, and the cost arrives
weeks later on the sending domain.

So two paths are written down as evaluation fixtures, the two where a regression is
both silent and expensive. Each case is a single line of JSON recording behaviour the
code already has.

- `suppression_classify.jsonl`, recognising an opt-out in an inbound reply
- `reply_triage_classify.jsonl`, sorting positive, opt-out, bounce and out-of-office

Those fixtures are held in the repository and no runner reads them yet, so they are a
record rather than a gate. What does run on every push and pull request is a test
suite covering the same class of risk: the suppression gate, rate limiting, dashboard
auth tokens, booking slot logic, a browser security header the live demo depends on,
and what the service does when a model call fails outright.

Chosen deliberately over a coverage number. These are the ones that fail without
telling you.

## A video pipeline, and the fork that proved it

[github.com/visione4906/durable-media-pipeline](https://github.com/visione4906/durable-media-pipeline).
Public, MIT licensed.

A channel needs output on a schedule, and by hand that costs hours per video. This
runs the whole chain: seed title, written script, narration, word-level timing,
generated images, assembly, then a scheduled upload. Progress is stored at each step,
so a failure resumes where it stopped instead of starting over.

Nothing publishes silently. Every video is sent to me on Telegram with the slot it is
scheduled for, and I can cancel it before it goes out. A single bad title is a strike
against the channel, and that is the one step worth keeping a person on.

Six scheduled tasks run it, all confirmed active on 2 August 2026, and it has
published real videos to a real channel.

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

## brain-engine

[github.com/visione4906/brain-engine](https://github.com/visione4906/brain-engine). Public.

A case-based decision engine in Rust. You describe a situation on closed vocabularies
and it returns one action and an expectation. When the outcome arrives, it scores the
case and closes it. Markdown stays the source of truth: SQLite holds document metadata
and span offsets, Tantivy holds the span text, and the index is derived and rebuildable.

Two decisions in it are worth more than the feature list. The discard log is part of
the answer, so `match` and `propose` return what they threw away and why. A matcher
that silently drops candidates is indistinguishable from one that had none, and the
difference is the whole point. And it refuses: `close` will not score an expectation
that has not come due yet, with a holdout keeping a tripwire on the refusal so the
refusal itself stays honest.

`mcp/server.py` wraps the compiled binary and exposes it to any MCP-capable client as
eight tools over stdio. It returns the engine's text verbatim, discard log and caveats
included, and does not summarise or re-rank what the engine said.

The corpus it reasons over is private and is not in the repository. What ships is the
engine and a pair of synthetic fixtures, so the loader and the validator can be tested
without anybody's business in the repo. Forty-nine tests pass; three are marked
ignored because they check parity against a Python reference that lives beside the
private corpus, and they are marked rather than deleted so the gap stays visible.

---

## Contact

[LinkedIn](https://www.linkedin.com/in/oladapo678). Open to AI engineering and AI
automation roles, remote.

More public repositories are on
[my profile](https://github.com/visione4906?tab=repositories).
