# Work Log

## Nov 8, 2026 — Comparing authentication methods

### What was done
- Wrote a post comparing password-based, token-based, and multi-factor authentication, and arguing which approach is most effective for enterprise systems.
- The angle: MFA wins, but not just any MFA — the kind of second factor matters, and the real test is what happens when one factor gets stolen. Passwords fail because of people (reuse, phishing, resets); tokens fix some of that but move the risk to the token; MFA adds a safety net, but SMS and push approvals have real weaknesses, and hardware keys and authenticator apps are the stronger forms.
- Posted the post the same day. I was the first poster — no classmates had posted yet, so the discussion was empty until then.
- Built this repo around the post: a beginner-friendly README that explains each method in plain English, with a comparison table, a glossary, and further reading.

### What went well
- The argument had a clear structure: why passwords fail, what tokens fix and what they do not, why MFA wins, and why the second factor matters. Each section builds on the one before it.
- The post stayed distinct from earlier write-ups on DevSecOps, the CIA triad, enterprise security, and Git/CI-CD — the focus here is authentication specifically, with examples (SIM swap, push fatigue, hardware keys checking the domain) that did not appear in the other posts.

### Challenges
- Keeping the post honest about MFA: it is tempting to say "just turn on MFA" and stop there, but the push-fatigue problem and the SMS weakness are real, and ignoring them would have made the post weaker. The fix was to argue for phishing-resistant MFA specifically, not MFA in general.
- Balancing length: the post itself is short, but the README needed enough depth to explain hashing, salts, JWTs, refresh tokens, and TOTP without turning into a textbook. The comparison table and glossary carry a lot of that weight so the main text can stay readable.

### Status
- Post: submitted, 1/1. Reply: still pending — no classmates had posted yet, so there was nobody to reply to. Reply due Dec 5.
