# FraserLink

You're helping me build FraserLink, a small software business I'm starting.

## What FraserLink is
FraserLink sells a missed-call text-back and Google review system to local trades and home-service businesses (plumbers, roofers, electricians, contractors) in Surrey and Delta, BC, Canada. I'm a solo 3rd-year SFU Computing Science student. Right now I'm building a working demo to show business owners, then running free 30-day pilots with my first 3 clients. Keep things simple. Don't over-engineer.

## How it works
1. A customer calls the business and nobody answers.
2. The call reaches our Twilio number. We play a short greeting, record a voicemail, and text the caller within seconds, e.g. "Hi, it's {BusinessName}. Sorry we missed your call, we're on a job. What can we help with? Reply here and we'll get back to you shortly."
3. We text the owner the caller's number plus a voicemail link/transcript.
4. Customer replies to our number are forwarded to the owner as alerts, e.g. `604-555-0123 says: "My water heater is leaking." Call them: 604-555-0123`. This is one-way: the owner never replies through our number. They call or text the customer from their own phone, so each customer gets their own normal thread and nothing gets mixed up.
   - On a customer's first reply, we send one acknowledgment, e.g. "Thanks! {OwnerName} from {BusinessName} will call you shortly from {OwnerCell}." (People ignore unknown numbers; naming the number gets the callback answered.)
   - If the owner texts our number anything that isn't a command, we reply: "This number can't message customers. Tap their number to call or text them."
5. After a job, the owner marks it done by texting "DONE" plus the last 4 digits of the customer's number (e.g. "DONE 0123"). Full numbers in any format also work. If two recent customers share the last 4 digits, we ask which one. Alerts end with a hint like "Job done? Text DONE 0123". The customer gets a thank-you text with the business's Google review link, plus one reminder 48 hours later unless they opted out.
6. Monthly numbers per business: calls caught, texts sent, customer replies, review requests sent. Text them to the owner once a month (e.g. "October: 14 missed calls caught, 9 customers replied, 6 review requests sent"). Owners never need to log in; the dashboard is mainly for me.

## Owner experience (keep it minimal)
Owners aren't technical. They keep their phone, their number, and how they answer calls. No app, no login.
- One-time onboarding (~10 minutes, I do it with them on their phone): dial the forwarding code, turn off iPhone Live Voicemail (and similar call-screening features on Android, like Pixel Call Screen), save our number as a contact (e.g. "Customer Texts"), set up their "don't text" list, and make a test call together.
- Day to day, all they need to know: missed-call alerts arrive by text, tap a number to call or text the customer, and text DONE when a job is finished.
- Turning it off is one code (##004#), so pilots are low-risk.
- Leave them a one-page cheat sheet (kept in the README).

## Call routing (support both per business, via a routing_mode setting)
- forward_mode (default for pilots): the owner keeps their number and sets conditional call forwarding on their phone (**61*number# for no answer, **004*number# for all conditional; ##61# / ##004# to turn off). Calls reaching Twilio were already missed, so greet, record and text right away. Log Twilio's ForwardedFrom parameter when present. Use **004* by default: unlike **61*, it also catches calls the owner declines and calls when they have no signal (common on roofs and in basements).
- direct_mode: our Twilio number is the main line. <Dial> the owner's cell with a ~20 second timeout and an action URL. On DialCallStatus no-answer, busy or failed, run the missed-call flow. Gotcha: the owner's carrier voicemail can "answer" first, so keep the timeout short or add call screening ("press 1 to take this call"). Supported in code but not offered in pilots: it would mean changing the number on their Google profile, truck and business cards.

## Edge cases
- Blocked/anonymous caller ID or a landline caller: can't text, but still notify the owner and save the voicemail.
- Same caller calling several times in a few minutes: send only one text-back.
- No review texts at night: queue anything between 9pm and 9am (America/Vancouver) for the next morning.
- Personal calls: many owners use one cell for everything. For numbers on the business's "don't text" list (family, friends, suppliers), skip the text-back but still save the voicemail and notify the owner.
- A customer calls back our Twilio number (the one that texted them): it must ring the owner, not loop into the missed-call flow or create a forwarding loop. Solve this in Milestone 1.

## Rules we must follow
- CASL (Canadian anti-spam law): log the consent basis and timestamp for every contact (inquiry or purchase). Honor STOP/START/HELP and store opt-outs in our own DB too.
- Google review policy: send the same review request to every customer. No "review gating" (no rating-first screens that route unhappy customers away) and no incentives for reviews.
- Twilio: trial accounts can't run custom TwiML or custom messages, so we'll use an upgraded pay-as-you-go account. New Canadian numbers likely need messaging registration before texting real customers; flag this as a setup task.
- Validate Twilio request signatures on every webhook. No secrets in the repo; use env vars.

## Tech defaults (suggest changes only with a good reason)
- Node.js + TypeScript, Express, PostgreSQL (Supabase is fine), Twilio SDK, ngrok for local webhook testing.
- Keep all Twilio-specific code in one telephony module behind a small interface, so we can switch providers (e.g. Telnyx) later.
- Multi-tenant from the start: each business has its own Twilio number. Tables: businesses (name, owner_name, twilio_number, owner_cell, google_review_link, greeting, message templates, routing_mode, timezone), contacts, calls, messages, review_requests, consent_log, plus each business's "don't text" list.
- Message templates are editable per business.
- Write tests for webhook handlers.

## Milestones
1. Missed-call flow for one business (config via env, duplicate-call check kept in memory): greeting, voicemail, text-back, owner notification. This is the demo.
2. Move to Postgres with the full multi-tenant tables (first business seeded from env), so alerts, opt-outs, consent records and reminders survive restarts. Then forward customer replies to the owner as alerts, send the one-time acknowledgment text, and handle owner commands (DONE).
3. Review requests with the 48-hour reminder and quiet hours.
4. Onboard more businesses (each with its own Twilio number), the monthly summary text to owners, and a minimal dashboard for me: leads, calls, review requests, monthly numbers.
5. Deploy with HTTPS (Render, Railway or Fly).

Out of scope for now: AI receptionist, Jobber integration, payments, number porting.

## How to work with me
I'm learning as I go. Explain key decisions in a sentence or two. Ask before adding any paid service or new dependency. Keep a README with setup steps: Twilio console config, ngrok, phone forwarding codes, and turning off Live Voicemail on iPhone for testing. Include the owner onboarding checklist and the one-page owner cheat sheet.
