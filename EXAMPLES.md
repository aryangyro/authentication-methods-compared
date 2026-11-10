# Worked examples: picking an authentication method

Three short scenarios. Each one picks a different method, for a
different reason — the method is never "the best one", it is the one
that fits the risk.

## 1. A small online shop

**Situation:** A shop with a few thousand customers. People log in to
order, save an address, and pay. The owners are two people and there
is no IT team.

**Choice:** Passwords plus a one-time code by email for login, and a
short-lived token while the customer is browsing.

**Why:** Passwords alone are too weak — credential stuffing would
empty accounts. But hardware keys are too much to ask of casual
shoppers. A code by email is a middle ground people actually use.
Tokens keep the session safe without asking for the password on every
page, and short expiry limits the damage if one is stolen.

## 2. A hospital's internal records system

**Situation:** Doctors and nurses open patient records all day, from
shared computers on wards. A stolen login could expose medical
records, which are worth a lot to attackers.

**Choice:** Password plus a hardware key or authenticator app, with
tokens that expire quickly and sessions that lock when the computer
is idle.

**Why:** This is the case where friction is worth it. A code by SMS is
not enough — SIM swapping is a known attack, and the data is too
sensitive. Staff grumble about the extra step, but the alternative is
a breach that ends up in the news. Short sessions matter because the
computers are shared.

## 3. A hobby project with three users

**Situation:** A personal project, three users, no sensitive data —
just a to-do list and some notes.

**Choice:** Passwords, hashed properly, with account lockout after a
few wrong tries. No MFA, no tokens.

**Why:** Adding MFA here would cost more in setup and annoyance than
it saves in risk. What matters is the basics done right: never store
a plain-text password, lock out guessing, and log failed logins.
Security is about the risk in front of you, not the longest list of
features.
