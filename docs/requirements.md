Product summary and pilot scope
OPAL is a web app where members complete simple daily tasks for points, then redeem points for naira. The pilot keeps every user-facing feature but moves all real money outside the app: the app keeps the points ledger and request statuses, and admins send and confirm bank transfers by hand.
A user signs up, pays a one-time Plus activation by bank transfer, and an admin confirms it. Active members complete 12 daily tasks, take one daily spin and invite friends. They withdraw points once every 30 days if they meet the level minimum and have one qualified referral. Referral bonuses sit in a separate balance and are paid on two fixed days a week.
In the pilot
Not in the pilot
Sign-up with name, WhatsApp number, email, password
WhatsApp or SMS codes
Plus activation by bank transfer, confirmed by an admin
Card, USSD or any payment provider
Daily tasks, daily spin, points ledger
Task verification of any kind
Points withdrawal and referral withdrawal requests
Automatic payouts
Five levels: Plus, Pro, Max, Star, Senior
Paid upgrades between levels
Admin console with three roles
Automatic bank name lookup
Email code for password reset
Push, SMS or WhatsApp alerts sent by the system
Light mode, desktop and mobile web
Dark mode, native apps, tablet-specific layouts
Language is English. Currency is naira (₦). All times are West Africa Time (Africa/Lagos).
3. Tech setup
Build one responsive React app in Lovable on a Supabase backend, with the user app and the admin console in the same project under different routes.
Part
Choice
Notes
Front end
Lovable default: React, TypeScript, Tailwind, shadcn/ui
Restyle shadcn components to the tokens in section 4
Database and auth
Supabase project owned by OPAL
Do not use a Lovable-managed backend you cannot export
Email
Supabase Auth email through a custom SMTP sender (for example Resend)
Used for password reset codes and admin sign-in codes only
Server logic
Postgres functions and Supabase Edge Functions
Everything in section 10 runs here, never in the browser
Scheduled jobs
Supabase cron
Daily task reset at 12:00 AM WAT
Hosting
Lovable publish, custom domain later

Sign-in with a WhatsApp number. Supabase Auth signs users in by email. Store the real email in Supabase Auth and the WhatsApp number in profiles. The sign-in screen takes number and password, and an Edge Function looks up the email for that number and signs in. The error message is the same whether the number or the password is wrong.
Deliberately not integrated. No payment provider, no WhatsApp Business API, no SMS, no bank account name lookup. Links that say "WhatsApp" open https://wa.me/<number> with a prefilled message and nothing more.
Environments. Use a separate Supabase project for testing. Never test with the production database once real users exist.
4. Design system
Build light mode only, in Inter, using the tokens below as Tailwind theme values; never hard-code a colour in a component.
Colours
Token
Hex
Use
bg
#F5F7FB
Page background
surface
#FFFFFF
Cards, dialogs, inputs
sunk
#EDF0F6
Disabled, skeletons, tab tracks
line
#E1E6EF
Borders and dividers
ink
#0B1430
Primary text
ink2
#4A556C
Secondary text
ink3
#667085
Labels and helper text
primary
#0061FF
Filled buttons, active nav, progress, links
primaryHover / primaryPressed
#0052D9 / #0047BF
Button states
primarySoft
#EAF3FF
Selected rows, icon tiles
secondary
#0F1D40
Admin sidebar, auth brand panel, toasts
secondarySoft
#E7EBF3
Spin card
green / greenSoft
#177245 / #E3F4EA
Success, Paid, Qualified, Active
gold / goldSoft
#8A5F00 / #FAF0D7
Waiting, Processing, Awaiting confirmation
red / redSoft
#C8321E / #FBE9E5
Errors, Failed, Held
Type (Inter)
Style
Size / line height
Weight
Use
display/xl
48 / 110%
Semi Bold
Page greeting on desktop
display/lg
36 / 112%
Semi Bold
Page titles
display/md
26 / 120%
Semi Bold
Dialog titles
display/sm
20 / 128%
Semi Bold
Card titles
title
16 / 135%
Semi Bold
Row titles
body/lg, body/md, body/sm
16, 15, 13 / 150%
Regular
Body text; 15 is the default
label
12 / 135%, uppercase, +4% spacing
Medium
Field and table labels
button
14
Semi Bold
Buttons
chip
12
Medium
Status chips
num/xl, lg, md, sm
52, 30, 18, 14
Semi Bold
Balances and amounts, tabular figures
Headings use slightly negative letter spacing (about -2%).
Spacing and shape
• Spacing scale: 4, 8, 12, 16, 20, 24, 32, 40, 56, 64 px.
• Radius: 10 (chips, small tiles), 12 (inputs), 16 (cards), 20 (large cards), 24 (dialogs and sheets), full (buttons, chips).
• Buttons are pill shaped: 48 px high with 24 px side padding, or 40 px high with 18 px padding for the small size.
• Dialog shadow: 0 24px 48px at 10% black. Toast shadow: 0 8px 24px at 18% black. Cards use a 1 px line border and no shadow.
Components to build once and reuse
• Button: Primary, Secondary (outline), Soft, Accent (secondary fill), Ghost (text only), Disabled; two sizes.
• Chip: Neutral, Primary, Secondary, Gold, Green, Red, Outline. Always pair colour with a word.
• Input with label, helper text, error text and focus ring; six-box code input.
• Card, key-value row list with dividers, data table with header, tabs, toast (Success, Error, Info), note box with icon.
• Task row (desktop) and task card (mobile) with states Available, Claimable, Completed, Expired.
• User sidebar, mobile tab bar, admin sidebar.
Layout
• Desktop (1024 px and wider): fixed left sidebar about 260 px (248 px in admin), content up to about 1100 px wide.
• Mobile (below 1024 px, designed at 390 px): top bar with logo, bottom tab bar with Home, Tasks, Wallet, Profile, Referrals.
• Every desktop dialog (520 px wide, centred, dimmed page behind) becomes a bottom sheet on mobile with a grabber, 24 px top corners.
• Sign-up, sign-in and password reset use a split layout on desktop: navy brand panel left, form card right. On mobile the brand panel is dropped.
• Minimum touch target 44 px. Visible focus ring on every control. Dialogs trap focus and close on Escape.
5. Roles and access
There are two kinds of account, members and admins, and they never share a sign-in screen or a table of permissions.
Member states
State
How they get there
What they can do
Not activated
Signed up
See Home, the locked task list, How OPAL works, Profile, Settings, Referrals (share link). Cannot claim tasks, spin or withdraw.
Awaiting confirmation
Tapped "I've sent ₦5,000"
Same as not activated, with "We're confirming your payment" on Home, Tasks and Wallet.
Active (Plus to Senior)
Admin confirmed the activation payment
Everything.
Suspended
Admin suspended the account
Sign in and see the suspended screen only. Balances are held, not removed.
Admin roles
Capability
Owner
Operations
Viewer
View every admin screen
Yes
Yes
Yes
Confirm activation payments
Yes
Yes
No
Process withdrawals
Yes
Yes
No
Create and edit tasks
Yes
Yes
No
Approve or reject flagged referrals
Yes
Yes
No
Suspend users, change a user's number, reset a user's password
Yes
Yes
No
Adjust points in the ledger
Yes
No
No
Change configuration
Yes
No
No
Invite, change or remove admins
Yes
No
No
• Admins are created by invitation only. The invite link lasts 7 days and sets name and password.
• There must always be at least one Owner.
• Controls a role cannot use are hidden. Opening a restricted URL directly shows "Admin / No permission".
• Every admin action that changes data writes a row to the audit log with who, what and when.
• Admin sign-in is work email and password, then a 6-digit code emailed to that address (10-minute expiry).
6. Data model
The points ledger is the single source of truth: every balance is the sum of ledger entries, and no screen or function edits a balance directly.
Table
Key fields
Notes
profiles
id (auth user), full_name, whatsapp_number (unique), email (unique), status, level, activated_at, cycle_start, narration_code (unique), referral_code (unique), referred_by, must_change_password
status: not_activated, awaiting_confirmation, active, suspended. level: plus, pro, max, star, senior.
activation_payments
user_id, amount, narration_code, status, reported_at, decided_at, decided_by, bank_reference
status: reported, confirmed, not_found. One row with status reported per user at a time.
tasks
slug, title, description, category, reward_points, link_url, active
link_url is empty for the daily login reward.
task_instances
user_id, task_id, cycle_date, state, opened_at, claimed_at
state: available, claimable, completed, expired. Unique on user, task, cycle_date.
spins
user_id, cycle_date, points
Unique on user and cycle_date.
ledger_entries
user_id, balance, type, amount, balance_after, related_id, created_by, note, created_at
balance: points or referral. type: task_reward, spin_reward, referral_bonus, withdrawal, withdrawal_return, adjustment. Insert only; never updated or deleted.
referrals
referrer_id, referred_id (unique), status, bonus_points, qualified_at, flag_reason
status: pending, qualified, flagged, rejected.
payout_accounts
user_id, bank_name, account_number, account_name, is_default
Typed by the user.
withdrawals
reference, user_id, source, points, rate, gross_naira, tax_rate, tax_naira, fee_naira, net_naira, account snapshot, status, payout_date, locked_by, locked_at, bank_reference, failure_reason, requested_at, paid_at, paid_by
source: points or referral. status: requested, scheduled, processing, paid, failed, cancelled, reversed.
levels
name, position, min_withdraw_points, referral_bonus_points
Five rows, editable by an Owner.
config
conversion_rate, plus_price, fee_naira, tax_rate, referral_min_points, payout_days, payout_cutoff, activation bank name, account number, account name, support_whatsapp, max_not_found_reports
One row.
admins
user_id, role, status
role: owner, operations, viewer.
admin_invites
email, role, token, expires_at, accepted_at

audit_log
admin_id, action, target, before, after, created_at
Insert only.
References use readable formats: WD-YYYYMMDD-XXXX for points withdrawals, RW-YYYYMMDD-XXXX for referral withdrawals, OPAL-XXXX for narration codes (random letters and digits, never taken from the phone number), USR-NNNNN for users.
7. Business rules
Every number below is read from config or levels at run time; the values shown are the current settings.
Sign-up and sign-in
• Sign-up fields: full name (two or more words), WhatsApp number (+234 and 10 digits, unique), email (valid, unique), password (8 or more characters with a number). The button stays disabled until all four are valid.
• No code is sent at sign-up. "Create account" opens "Is this your WhatsApp number?" showing the number back. Confirming creates the account.
• A referral link /r/<code> stores the referrer at sign-up. It cannot be changed afterwards.
• After sign-up, show "How OPAL works" once, then Home in the not-activated state.
• Sign-in: WhatsApp number and password. Wrong details show one combined message. Sign-in pauses for 15 minutes after 5 wrong attempts.
• Password reset: email, 6-digit code by email (10-minute expiry, resend after 30 seconds, 3 wrong codes pause for 15 minutes), new password. The code screen shows the same message whether or not the email has an account. Changing the password signs out other devices.
• If must_change_password is true, force the new-password screen straight after sign-in.
Activation
1. The user opens "Activate / Confirm Plus", then "Pay by bank transfer", which shows the OPAL bank details from config, the amount (₦5,000) and the user's narration code.
2. "I've sent ₦5,000" creates an activation_payments row with status reported and sets the profile to awaiting_confirmation. Nothing is unlocked.
3. An admin confirms it in "Admin / Activation payments" after finding the transfer on the bank statement, and enters the bank reference.
4. Confirming sets the profile to active at level Plus, sets activated_at and cycle_start to now, and qualifies the referral (see Referrals), all in one transaction.
5. "Mark as not found" sets the payment to not_found and the profile back to not_activated. The user sees "Payment not found" and can report again.
6. After 3 not-found reports, "I've sent ₦5,000" is disabled and the user is told to contact support on WhatsApp.
Levels
Level
Minimum to withdraw
Referral bonus
Plus
50,000 pts
5,000 pts
Pro
300,000 pts
10,000 pts
Max
600,000 pts
15,000 pts
Star
1,000,000 pts
20,000 pts
Senior
2,000,000 pts
30,000 pts
• All levels have the same features. Only the two numbers change.
• A member moves up one level each time a points withdrawal is marked as paid. It is free and automatic. Show "Membership / Pro unlocked" (or the matching level) the next time they open the app.
Points withdrawal
• Eligible when all are true: profile is active; today is on or after cycle_start + 30 days; no points withdrawal has been requested in this cycle; points balance is at least the level minimum; the member has at least 1 qualified referral; a payout account is saved.
• The referral requirement stays met for good once it has been met.
• The amount is any whole number of points from the level minimum up to the full balance.
• Payout maths: gross = floor(points ÷ 5). Tax = 10% of gross. Payout = gross − tax − ₦50 fee. Example: 50,000 pts gives ₦10,000 gross, ₦1,000 tax, ₦50 fee, ₦8,950 paid.
• Rate, tax and fee are shown before the user confirms and are stored on the withdrawal at confirmation. Later config changes do not alter it.
• Confirming deducts the points with a ledger entry, sets status requested and starts a new cycle from that moment. The next window opens 30 days later.
• If a withdrawal fails or is rejected, the points return with a ledger entry and the window stays open.
Withdrawal statuses
1. Requested: created by the user, waiting for an admin.
2. Scheduled: referral withdrawals only, waiting for the payout day.
3. Processing: an admin tapped "Start paying". The request is locked to that admin; other admins see who holds it and cannot act. "Release without paying" returns it to the previous status.
4. Paid: the locking admin entered a bank transfer reference. A paid request can never be paid again. The user sees the reference.
5. Failed: the admin marked it failed with a reason. Points return.
6. Cancelled: the user cancelled a scheduled referral withdrawal before the cut-off. Points return.
7. Reversed: the bank returned the money after payment; an Owner reverses it. Points return.
Referrals
• Signing up with a link creates a referral with status pending. It shows to the referrer as "Signed up".
• While the friend's payment is awaiting confirmation, show "Payment being confirmed".
• When an admin confirms the friend's activation, the referral becomes qualified and the bonus for the referrer's current level is added to the referrer's referral balance with a ledger entry.
• If the friend shares a device or a payout account with the referrer, the referral is flagged instead. It pays nothing and does not count until an admin approves it.
Referral balance withdrawal
• Separate from the points balance. No 30-day window, level minimum or referral requirement.
• Needs an active profile, a payout account and at least 5,000 pts. One open request at a time.
• Same payout maths as above. Example: 10,000 pts gives ₦2,000 gross, ₦200 tax, ₦50 fee, ₦1,750 paid.
• Payout days are Tuesday and Friday with a 12:00 PM WAT cut-off. A request made before the cut-off on a payout day is paid that day; otherwise on the next payout day. Always show the payout date before the user confirms.
• Points leave the referral balance at confirmation. The user can cancel until the cut-off on the payout day.
Daily tasks
• 12 tasks, the same set every day. A new set of instances is created for every active member at 12:00 AM WAT.
• "Start task" opens the link in a new tab and sets the instance to claimable. Nothing checks what the user did there.
• "Claim pts" sets it to completed and adds the reward with a ledger entry straight away. Show a success toast.
• The daily login reward has no link and is claimable as soon as the day starts.
• Unfinished instances expire at reset. Rewards do not carry forward.
• Countdowns use server time.
Daily spin
• One free spin per member per day. Eight equal slices: 500, 1,000, 1,500 and 2,500 pts, each twice. The result is chosen on the server.
• The prize is added with a ledger entry straight away.
Messages to users
• The system sends two kinds of email only: password reset codes and admin sign-in codes.
• Admins send two WhatsApp messages by hand: account activated, and withdrawal paid. The admin dialogs remind them to.
8. User app: screens and routes
The user app has 12 routes; everything else is a dialog on desktop and a bottom sheet on mobile.
Route
Figma frames
States to build
/signup, /r/:code
Sign up; Sign up / Invited by a friend; Sign up / Validation errors; Sign up / Confirm WhatsApp number
Empty, invited banner, field errors (number in use, email in use, short password), confirm-number dialog
/welcome
Welcome / How OPAL works
Shown once after sign-up; reachable again from Profile
/signin
Sign in; Sign in / Wrong details; Sign in / Paused after too many attempts
Default, wrong details with attempts left, paused with countdown
/reset-password
Forgot password / Enter email, Enter code, Incorrect code, Code expired, New password, Passwords do not match, Password changed
Three steps plus the error states
/ (Home)
Home; Home / Account not activated; Home / Awaiting payment confirmation; State / Home loading
Active dashboard, not activated, awaiting confirmation, loading skeleton
/tasks
Tasks; Tasks / Awaiting payment confirmation; State / Tasks all completed; State / Tasks could not load; Daily reset
Ready to claim, available, completed lists; locked; all done; load error; reset notice
/wallet
Wallet; Wallet / Awaiting payment confirmation; State / Wallet with no transactions; Withdrawal status cards (six variants)
Balance card with the right status variant, referral balance card, transaction history with filters, empty
/referrals
Referrals; Referrals / No referrals yet
Link and code, requirement card, counts, referral balance, list with statuses Signed up, Payment being confirmed, Qualified, Not counted
/membership
Membership levels
Current level, progress to next withdrawal, table of five levels
/spin
Daily spin / Before; Daily spin / After
Ready, spinning, result, already spun today
/profile
Profile
Name, phone number, member since; membership summary; balances
/settings
Settings
Contact details (number, email), Membership, Updates, Security, Payout accounts
Dialogs and sheets
Group
Figma frames
Activation
Activate / Confirm Plus; Pay by bank transfer; Awaiting confirmation; Payment not found; Plus active
Points withdrawal
Withdraw / Requirements not met; Needs a referral; Window open; No payout account; Add payout account; Enter amount; How your payout is calculated; Review; Processing; Paid; Failed; Transaction detail
Referral
Referral / Share; Friend signed up; Qualified; Not counted
Referral withdrawal
Referral withdrawal / Enter amount; Enter amount, after cut-off; No payout account; How your payout is calculated; Review; Scheduled; Cancel request; Paid; Failed; Nothing to withdraw
Membership
Membership / Pro unlocked (reuse for every level)
Account
Account / Edit profile; Change password; Change WhatsApp number; WhatsApp number changed; Sign out; Sign out other sessions; Remove payout account
System
System / Network failure; Session expired; Account suspended
Behaviour notes
• The "Withdraw" button on Wallet opens the first dialog whose condition applies: requirements not met, needs a referral, no payout account, or window open.
• "Change WhatsApp number" asks for the new number and the current password. No code is sent.
• "Edit profile" edits name and email.
• "Add payout account" takes bank (dropdown of Nigerian banks), 10-digit account number and account name typed by the user.
• Copy buttons (link, code, account number) show a toast.
• Task claims, spin results and saved settings confirm with a toast, not a dialog.
• Mobile frames are named "Mobile / …" and "Mobile sheet / …" and match the desktop frames one to one.
9. Admin console: screens and routes
The admin console lives under /admin, is desktop only, and has nine sidebar items.
Route
Figma frames
What the admin does
/admin/signin
Admin / Sign in; Sign in error; Sign-in code; Accept invite; Invite expired
Email and password, then emailed code. Accept an invite to set name and password.
/admin
Admin / Overview
See today's numbers, the "Needs attention" queue (activation payments, withdrawals requested, failed withdrawals), points liability and membership counts.
/admin/users
Admin / Users; User detail; Change user WhatsApp number; Reset user password
Search and filter users. On a user: adjust points (Owner), suspend, change WhatsApp number, create a temporary password.
/admin/tasks
Admin / Tasks; Task editor
Create, edit, activate or deactivate tasks. Fields: title, description, category, reward points, task link, active. Changes apply from the next daily cycle.
/admin/activations
Admin / Activation payments; Confirm activation payment
Tabs: Awaiting, Confirmed, Not found. Confirm with a bank reference, or mark as not found.
/admin/withdrawals
Admin / Withdrawals; Withdrawal review; Withdrawal review / Paying; Withdrawal review (referral balance) / Paying
Tabs by status. Review shows the payout maths, the full payout account and the eligibility checks. Start paying, then Mark as paid with a bank reference, Mark failed, or Release.
/admin/withdrawals/referral-batch
Admin / Referral payout batch; Referral payout list
See the next payout day's batch, pay each request and mark it paid. Held requests are reviewed separately.
/admin/referrals
Admin / Referrals
List referrals. Approve or reject flagged ones.
/admin/ledger
Admin / Ledger; Administrative adjustment
Read every ledger entry. An Owner can add a credit or debit with a reason and a note.
/admin/configuration
Admin / Configuration; Edit conversion rate; Edit referral payout days
Owner only. Edit every value in config and levels. Each save needs a reason and is written to the audit log.
/admin/team
Admin / Team; Invite admin; Change role; Remove admin; No permission
Owner only. Invite, change role, remove.
Behaviour notes
• Table rows are sorted oldest first in queues, so the longest wait is on top.
• The Withdrawals table shows the amount to send (after tax and fee), not the gross value.
• Eligibility is recalculated on the server when a review dialog opens. A request that no longer passes shows the failing check in red and cannot be started.
• The confirm-activation and mark-as-paid buttons stay disabled until a bank reference is typed.
• "Export" buttons download the current table as CSV.
• Sample data in Figma (names, counts, amounts) is illustrative. Build every number from the database.
10. Security rules
The browser may read a member's own data and call named server functions; it must never write points, statuses or balances directly.
Row-level security
• Turn on row-level security for every table.
• A member can read only their own rows in profiles, task_instances, spins, ledger_entries, referrals (as referrer), payout_accounts, withdrawals and activation_payments.
• A member can update only their own full_name and email in profiles, and insert or delete their own payout_accounts.
• Members have no insert, update or delete rights on ledger_entries, withdrawals, task_instances, spins, referrals, activation_payments, config, levels or any admin table.
• Admin tables and admin reads are allowed only when the caller has a row in admins with status active. Role checks happen inside each function.
• The Supabase service key is used only inside Edge Functions and is never sent to the browser.
Server functions (the only way to change money-related data)
Function
Caller
What it must check and do
report_activation_payment
Member
No open report; under the not-found limit. Creates the report, sets status awaiting_confirmation.
open_task
Member
Profile active; instance is available today. Sets claimable.
claim_task
Member
Instance is claimable today. Sets completed and writes the reward entry, in one transaction, exactly once.
spin
Member
Profile active; no spin today. Picks the slice on the server and writes the entry.
request_points_withdrawal
Member
Re-checks every eligibility rule and the amount. Computes the payout from config, writes the deduction and the withdrawal, starts the new cycle.
request_referral_withdrawal, cancel_referral_withdrawal
Member
Minimum, payout account, no open request; computes the payout date. Cancel only before the cut-off.
change_whatsapp_number
Member
Verifies the current password; number is unique.
confirm_activation, mark_activation_not_found
Owner, Operations
Payment is still reported. Confirm activates, starts the cycle and qualifies or flags the referral in one transaction.
start_paying, release_withdrawal
Owner, Operations
Start only if not locked and still eligible. Release only by the locking admin or an Owner.
mark_withdrawal_paid, mark_withdrawal_failed
Owner, Operations
Caller holds the lock; status is processing; bank reference present for paid. Paid moves the member up one level. Failed returns the points.
review_flagged_referral
Owner, Operations
Approve credits the bonus once; reject closes it.
admin_change_user_number, admin_reset_user_password, suspend_user
Owner, Operations
Writes the audit log. Reset sets must_change_password.
adjust_points, update_config, manage_admins
Owner
Reason required. Writes the audit log.
Rules for every function
• Run the checks and the writes in one database transaction, with a row lock on the member's profile, so two taps or two tabs cannot pay twice.
• Recalculate from the ledger on the server. Never trust an amount, balance or status sent by the browser.
• A ledger entry is written once per event. Use a unique key (for example task instance id, withdrawal id) so a retry cannot create a second entry.
• Rate-limit sign-in, password reset and report-payment by account and by IP address.
• Passwords are handled by Supabase Auth only. Never store or log them. A temporary password is shown to the admin once and not stored in plain text.
