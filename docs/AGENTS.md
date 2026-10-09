OPAL v2 Pilot: project knowledge

The full requirements are in docs/requirements.md in this repository. Read the relevant section of that file before building or changing any feature. If this summary and that file disagree, the file wins. Figma frame names quoted in prompts match the Figma page "v2 Pilot".

PRODUCT
OPAL is a responsive web app. Members complete 12 daily tasks for points, take one daily spin, invite friends, and redeem points for naira. In this pilot no real money moves through the app: the app keeps a points ledger and request statuses, and admins send and confirm bank transfers by hand. English, naira, Africa/Lagos time.

NOT IN THE PILOT (never add)
Payment providers, card or USSD checkout, automatic payouts, WhatsApp or SMS codes, system-sent WhatsApp/SMS/push alerts, bank account name lookup, task verification, dark mode, native apps.

STACK
React, TypeScript, Tailwind, shadcn/ui. Supabase project owned by OPAL for database, auth, Edge Functions and cron. User app and admin console in one project; admin under /admin, desktop only.

DESIGN
Light mode only. Font Inter. Use theme tokens, never hard-coded colours: bg #F5F7FB, surface #FFFFFF, sunk #EDF0F6, line #E1E6EF, ink #0B1430, ink2 #4A556C, ink3 #667085, primary #0061FF (hover #0052D9, pressed #0047BF), primarySoft #EAF3FF, secondary #0F1D40, green #177245 / #E3F4EA, gold #8A5F00 / #FAF0D7, red #C8321E / #FBE9E5. Buttons are pills, 48px or 40px high. Radius: inputs 12, cards 16, dialogs 24. Cards have a 1px line border and no shadow. Desktop from 1024px: left sidebar. Below 1024px (designed at 390px): top bar and a 5-tab bottom bar (Home, Tasks, Wallet, Profile, Referrals), and every dialog becomes a bottom sheet. Status chips always pair colour with a word.

ACCOUNTS
Member states: not_activated, awaiting_confirmation, active, suspended. Levels: plus, pro, max, star, senior. Admin roles: owner, operations, viewer. Admins are invited, sign in at /admin/signin with email, password and an emailed 6-digit code. Viewer changes nothing. Only an owner adjusts points, changes configuration or manages admins. Every admin change writes to audit_log.

SIGN-UP AND SIGN-IN
Sign-up fields: full name, WhatsApp number (+234, unique), email (unique), password. No code at sign-up; show "Is this your WhatsApp number?" before creating the account. Sign-in is WhatsApp number plus password (an Edge Function looks up the email). Password reset uses a 6-digit email code.

ACTIVATION
The member pays the Plus price by bank transfer to the OPAL account shown from config, quoting their unique narration code (OPAL-XXXX, random). "I've sent it" only creates a report and sets awaiting_confirmation. An admin confirms it with a bank reference; that activates the member at Plus, starts the 30-day cycle and qualifies the referral, in one transaction. "Not found" returns the member to not_activated; after 3 not-found reports the report button is disabled.

MONEY RULES (read from config and levels, never hard-code)
Conversion 5 pts = 1 naira. Payout = floor(points / 5) minus 10% tax minus 50 naira fee. Example: 50,000 pts pays 8,950 naira. Level minimums to withdraw: 50,000 / 300,000 / 600,000 / 1,000,000 / 2,000,000 pts. Referral bonuses: 5,000 / 10,000 / 15,000 / 20,000 / 30,000 pts.
Points withdrawal needs all of: active profile, 30 days since cycle start, no withdrawal yet this cycle, balance at or above the level minimum, at least 1 qualified referral, a saved payout account. A paid points withdrawal moves the member up one level.
Referral bonuses go to a separate referral balance. Referral withdrawals need only an active profile, a payout account and 5,000 pts; they are paid on Tuesday and Friday with a 12:00 PM cut-off and can be cancelled before it.
Withdrawal statuses: requested, scheduled, processing, paid, failed, cancelled, reversed. An admin taps Start paying (locks the request to them, status processing), sends the money outside the app, then marks it paid with a bank reference. A paid request can never be paid again. Failed, cancelled and reversed return the points.

TASKS AND SPIN
Start task opens the link in a new tab and makes the task claimable. Claim pts adds the reward immediately. Nothing is verified. Tasks reset at 12:00 AM Africa/Lagos; unfinished ones expire. One spin per day; the server picks one of 8 equal slices (500, 1,000, 1,500, 2,500 pts, twice each).

NON-NEGOTIABLE ENGINEERING RULES
1. ledger_entries is the source of truth. Balances are the sum of ledger entries. Ledger rows are insert-only.
2. The browser never writes points, balances, statuses, withdrawals or config. Those change only through the named server functions in docs/requirements.md section 10.
3. Every server function re-checks eligibility on the server, runs in one transaction with a lock on the member's profile, and is safe to retry (one ledger entry per event).
4. Row-level security is on for every table. Members read only their own rows. Admin access requires an active row in admins, and the role is checked inside each function.
5. The Supabase service key is used only in Edge Functions.
6. Sample names and numbers in Figma are illustrative; all data comes from the database.
7. Change only what the prompt asks for. Do not refactor or restyle working screens.
