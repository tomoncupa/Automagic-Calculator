# Automagic Fitness Plan — lead magnet

A single-file, no-dependency web page. Visitor enters twelve fields of biodata; the page
computes energy needs, macros and a full training week client-side. Day 1 and all the
nutrition numbers are free. Days 2+, weekly set volume, conditioning and the progression
rules unlock in exchange for an email.

`fitness-plan.html` is the whole product. No build step, no backend, no npm.

---

## 1. Wire it to your list

Open `fitness-plan.html`, find the `CONFIG` block near the top of the `<script>` (search for
`CONFIG = {`), and set `endpoint` plus `fields`.

| Provider | `endpoint` | `fields` | `opaque` |
|---|---|---|---|
| **Kit (ConvertKit)** | `https://app.kit.com/forms/<FORM_ID>/subscriptions` | `{email:"email_address", name:"first_name"}` | `true` |
| **Mailchimp** | your embedded form's `action` URL, with `/post` changed to `/post-json` | `{email:"EMAIL", name:"FNAME"}` | `true` |
| **MailerLite** | `https://assets.mailerlite.com/jsonp/<ACCOUNT>/forms/<FORM_ID>/subscribe` | `{email:"fields[email]", name:"fields[name]"}` | `true` |
| **Beehiiv** | your form's POST URL | `{email:"email", name:"first_name"}` | `true` |
| **Your own API** | your endpoint | whatever you accept | `false` |

To find the endpoint: in your provider, create the form, choose the **HTML / embed** option,
and copy the `action="..."` URL out of the markup.

**`opaque: true`** means the browser fires the request without being allowed to read the
reply (`mode: "no-cors"`). Almost every hosted form endpoint needs this, because they do not
send CORS headers. The consequence: the page cannot tell a success from a failure, so it
unlocks optimistically. That is the normal trade-off for embedded forms. Set `opaque: false`
only when you post to something you control that returns proper CORS headers — then a non-2xx
keeps the gate closed and shows an error.

Other `CONFIG` keys:

- `doubleOptIn` — `true` renders "check your inbox to confirm" after submit. Match it to how
  your list is actually configured, or the copy lies to people.
- `promise` — the one line stating what subscribing gets them. Shown on the form and again
  after unlock. Keep it accurate; it is the whole basis of the exchange.
- `privacyUrl` — optional. Set it and a Privacy link appears under the form.

### Verify it works

1. Set `endpoint`, save, open the file in a browser.
2. Submit a real address you control.
3. Confirm the subscriber appears in your provider's dashboard.

With `endpoint: ""` the page runs in **demo mode**: the gate unlocks, nothing is sent, and the
lead is logged to the browser console. That is the default so the file is safe to open and
click through before it is wired up.

---

## 2. Host it

It is one static file. Anywhere that serves static files works:

- Drop it into your site as `/fitness-plan.html`
- Netlify / Vercel / Cloudflare Pages — drag the file in
- GitHub Pages
- Embed in Webflow, Squarespace, Carrd, WordPress via an `<iframe>`

It needs a public URL to work as a lead magnet — that is the one thing it cannot be served
from as a Claude Artifact (see *Known limits*).

---

## 3. Compliance

You are collecting an email address with a stated purpose, which puts you under GDPR (EU/UK),
CAN-SPAM (US) and PECR (UK) depending on your audience. What is already built in:

- **Unticked consent checkbox**, required before submit. Never pre-tick it — pre-ticked
  consent is invalid under GDPR and it is the single most common failure in lead-magnet forms.
- **Purpose stated at the point of collection** — the plan, plus one email a week.
- **No pre-checked upsells, no hidden fields, no third-party sharing.**

What you still have to do:

- Honour unsubscribe in one click, in every email. Say so, then do it.
- Set `privacyUrl` to a real privacy policy naming you as data controller.
- Keep a record of consent — most providers timestamp this automatically; check yours does.
- If you have EU/UK subscribers, keep `doubleOptIn: true` and actually run double opt-in.
- Do not send anything outside what `promise` says without asking again.

---

## 4. What's free vs gated

Free, always, no email:

- Resting metabolic rate, maintenance, daily calorie target, BMI, expected rate of change
- Full macro split with per-meal protein, fibre, water and step targets
- Adherence guidance
- **Day 1 of the training week**, complete — exercises, sets, reps, RIR, rest
- The first stage of the "what to expect" timeline

Gated behind the email:

- Days 2 through N
- Weekly hard sets per muscle, against the 10–20 productive range
- Conditioning prescription and step target
- Progression, load increments and deload timing
- The week-by-week projection chart and stall-fixing rules

The free tier is deliberately a genuinely usable thing on its own. That is what separates an
ethical bribe from a bait-and-switch, and it is also what makes the email worth giving: people
who used Day 1 and found it credible convert far better than people who saw a blurred teaser.

**The gate is a courtesy gate.** The gated content is computed in the browser, so anyone who
opens dev tools can reach it. That is true of every client-side lead magnet, it is fine, and
trying to defeat it costs more than it returns. If you genuinely need the content unreachable,
you need a server that renders the full plan only after the email is verified.

---

## 5. Tuning the plan itself

All in `fitness-plan.html`:

- `LIB` — the exercise library. One entry per movement pattern, with a `gym` / `db` / `bw`
  variant each. Swap in your own exercise names here; nothing else needs to change.
- `DAYTPL` — day templates as ordered pattern lists. Seven patterns each; the session-length
  setting slices to 4, 5, 6 or 7.
- `buildWeek()` — split selection, set and rep prescription, RIR by experience and goal.
- `PACE` — rate of loss/gain as a percentage of body weight per week, by pace setting.
- `energy()` — equations, the 25% deficit cap and the calorie floor.
- `milestones()` — the week-by-week expectation copy, per goal.

### The numbers behind it

- BMR: Mifflin–St Jeor, or Katch–McArdle when a body-fat percentage is supplied
- TDEE: BMR × activity multiplier (1.2 – 1.9)
- Deficit capped at 25% of TDEE, with a floor of `max(BMR × 1.05, 1500 ♂ / 1200 ♀)`
- Under-18s are forced to maintenance with an on-page notice — no deficit, no surplus
- Protein 1.8–2.5 g/kg against lean mass where known, otherwise body weight capped at BMI 27.5
- Fat floored at 0.8 g/kg or 22% of calories, whichever is higher; carbs take the remainder
- 7,700 kcal/kg for fat loss, 5,500 kcal/kg for mixed tissue gain
- The projection recomputes BMR each week against the new weight, so the curve flattens the
  way real weight loss does instead of running in a straight line

---

## Known limits

- **It cannot run as a public Claude Artifact with real capture.** Artifacts that declare the
  `db` capability are organization-internal and cannot be shared publicly, and the Artifact CSP
  blocks `fetch` to third-party hosts — so neither artifact storage nor a POST to your email
  provider works there. The published Artifact is a demo-mode preview only. Host the file
  yourself for the real thing.
- Population-average equations. Individuals vary by roughly ±10% around them; the page says so.
- No exercise substitution for injuries or equipment gaps beyond the three equipment tiers.
