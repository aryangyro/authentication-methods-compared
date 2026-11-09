# Comparing Authentication Methods: Passwords, Tokens, and Multi-Factor Authentication

Every system that holds anything worth protecting has to answer one question before anything else happens: **who are you, and how do I know?** That is authentication. Get it wrong and nothing else matters — the strongest encryption in the world is useless if an attacker can simply log in as a real user.

This write-up compares the three authentication approaches most enterprises actually use: password-based authentication, token-based authentication, and multi-factor authentication (MFA). It looks at how each one works, where each one fails, and which one is most effective for an enterprise system — and why the answer is not quite as simple as "just turn on MFA".

If you are new to this, start at the top and read down. Every idea is explained in plain English before it is used.

## What authentication is (and what it is not)

Authentication is proving who you are. It is often confused with two related ideas, so it is worth separating them first:

- **Identification** is claiming an identity: "I am alice." A username is identification.
- **Authentication** is proving that claim: "I am alice, and here is my password to prove it."
- **Authorisation** is what you are allowed to do once you are in: "Alice may read patient records but not delete them."

This write-up is about the middle step — proving the claim — because that is where most real-world breaches start. Attackers rarely break encryption. Far more often, they log in with a stolen or guessed credential and walk through the front door.

Authentication is usually described in terms of **factors** — different kinds of proof:

| Factor type | What it means | Examples |
| --- | --- | --- |
| Something you know | Knowledge in your head | Password, PIN, security question answer |
| Something you have | A physical object or device | Phone, hardware security key, smart card |
| Something you are | A part of your body | Fingerprint, face, voice |
| Somewhere you are | A location | Office network, known IP address, GPS |

Password-based authentication uses one factor: something you know. Token-based authentication usually also starts with one factor, then issues a token that stands in for the login. Multi-factor authentication deliberately combines two or more factors, so that losing one is not enough to get in.

## Part 1: Password-based authentication

### How it works

Password-based authentication is the oldest and simplest approach. A user picks a secret string, the system stores a **hash** of it (never the password itself, if the system is built properly), and at login the user types the password, the system hashes what was typed, and compares the two hashes. If they match, the user is in.

A good password system also adds a **salt** — a random value mixed into the hash so that two users with the same password end up with different hashes, and so that pre-computed lists of hashes (rainbow tables) do not work. Modern password hashing is deliberately slow (bcrypt, Argon2, scrypt) so that guessing is expensive for an attacker even if the hash database leaks.

### Why passwords keep failing

A strong, unique password is fine on its own. The problem is people — and scale.

**Reuse.** In an enterprise with hundreds of employees, passwords get reused across systems. Once one password leaks in a breach somewhere else, attackers try it everywhere. This is called **credential stuffing**, and it works because people reuse passwords far more often than they admit.

**Phishing.** Hashing helps on the server side, but it does nothing if the user hands the password to a fake login page. A convincing phishing email, a lookalike domain, a fake "your password has expired" message — and the attacker has the real password, typed in by the real user. No amount of server-side hashing prevents that, because the user gave the secret away voluntarily.

**Writing them down.** Complex password rules (uppercase, lowercase, number, symbol, change every 30 days) often backfire. People cope by writing passwords on sticky notes, storing them in spreadsheets, or picking predictable patterns like `Summer2025!` followed by `Autumn2025!`. The policy meant to make passwords stronger made them weaker in practice.

**Resets.** Passwords also have a hidden cost: resets. Every reset is a support ticket, and every reset is another chance for someone to social-engineer their way in. "Hi, I'm locked out of my account, can you reset my password?" is one of the oldest attacks in the book, and it targets the help desk, not the login page.

**Guessing and brute force.** Short or common passwords fall to automated guessing. Lists of the most common passwords (`123456`, `password`, `qwerty`, the company name followed by `123`) are tried first, and they succeed depressingly often. Rate limiting and lockout help, but they are defences around the password, not fixes for it.

### What passwords are still good for

None of this means passwords are useless. They are cheap, everyone understands them, they need no special hardware, and they work as one layer of a stronger system. As a *first* factor, combined with something else, a password is fine. The mistake is treating a password as the *only* factor for anything important.

## Part 2: Token-based authentication

### How it works

Token-based authentication changes the shape of the problem. Instead of sending the password on every request, the user logs in once, and the server issues a **token** — a small piece of data that proves "this user already authenticated". The client then presents the token with each request instead of the password.

The best-known token format is the **JSON Web Token (JWT)**. A JWT has three parts: a header (what kind of token it is), a payload (who the user is, what they may do, when the token expires), and a signature (proof the token was really issued by the server and has not been tampered with). The server signs the token with a secret key, so anyone can read the payload but only the server can create a valid one.

Session tokens work similarly, but the server keeps a record of each session in a database instead of putting the data in the token itself. The trade-off is that session tokens need a database lookup on every request, while JWTs can be verified from the signature alone — but JWTs cannot easily be revoked before they expire, because there is no central list to check.

### What tokens fix

Tokens fix several real problems with passwords:

- **The password is not sent on every request.** It is sent once at login, then the token takes over. That shrinks the window in which the password can be intercepted.
- **Tokens can expire.** A token might last 15 minutes, an hour, or a day. Even if it is stolen, it stops working soon.
- **Tokens can be revoked.** If a device is lost or an account is compromised, the server can invalidate the token (for session tokens, by deleting the session; for JWTs, by keeping a blocklist or by using very short lifetimes with refresh tokens).
- **Tokens can carry limited permissions.** A token can say "this user may read, but not delete" — so even a stolen token is less useful than a stolen password, which usually gives full access.

### What tokens do not fix

But a token is only as safe as where it is stored. If it is stolen from a browser or a device, the attacker does not need the password at all — they just present the token. The server cannot tell a stolen token from a legitimate one, because the token *is* the proof.

This is called **token theft**, and it is a real attack. Tokens get stolen through cross-site scripting (XSS), through malware on the device, through man-in-the-middle attacks on unencrypted connections, and through careless logging (a token accidentally written into a log file is as good as leaked).

So token-based authentication moves the risk from the password to the token. It does not remove it. The token is a new secret, and it needs the same care as the old one: store it safely (httpOnly cookies rather than local storage, where JavaScript can read it), send it only over HTTPS, keep its lifetime short, and have a plan for revoking it.

A common design that gets the balance right is **short-lived access tokens plus refresh tokens**: the access token lasts minutes, the refresh token lasts longer but is stored more carefully and can be revoked, and the user re-authenticates with the refresh token to get a new access token. That way a stolen access token is useful for only a few minutes, and a stolen refresh token can be killed centrally.

## Part 3: Multi-factor authentication

### How it works

Multi-factor authentication (MFA) combines two or more factors from the list above. The classic combination is something you know (a password) plus something you have (a phone or a hardware key). The user logs in with the password, then proves the second factor, and only then gets access.

The logic is simple: an attacker who steals one factor still does not have the other. A phished password is not enough. A stolen phone is not enough without the password. Both have to fail at the same time, and that is much harder to arrange.

### Why MFA is the most effective for enterprise systems

That is why MFA is the most effective of the three for enterprise systems. It does not rely on one secret. Even if a password is phished, the attacker still needs the second factor, and even if a token is stolen, it usually expires fast and cannot be used to log in from scratch.

The other thing enterprises get from MFA is a kind of safety net. When one layer fails — and one layer will fail eventually — the other one still holds. A single point of failure becomes two points of failure that have to fail together. That is a much better position for a system with hundreds of users, any one of whom might click a phishing link on a bad day.

MFA also changes the economics for the attacker. Phishing a password is cheap and scalable: send a million emails, a few hundred people click. Phishing a password *and* a second factor, in real time, for a specific target, is expensive and hard to scale. Most attackers move on to easier targets.

### Not all MFA is equal

The key point, though, is that the second factor matters a lot. MFA is not a magic shield — it is only as strong as its weakest factor.

**SMS codes.** A code sent by text message is the weakest common second factor. It can be intercepted, and it can be defeated by a **SIM swap** — an attacker convinces the phone company to move the victim's number to a SIM card the attacker controls, and the codes start arriving on the attacker's phone. SMS was never designed as a security channel, and it shows.

**Push notifications.** A push approval ("Tap Yes to sign in") is more convenient, but it has a weakness called **push fatigue** or **MFA fatigue**. An attacker who already has the password triggers login after login, flooding the user's phone with approval requests. A tired user, woken up at 2am, eventually taps "Approve" just to make the notifications stop — and the attacker is in. This is exactly how some real, high-profile breaches happened. The attacker does not need to break the cryptography; they just need the user to get tired.

**Authenticator apps.** Time-based one-time passwords (TOTP) from an app like Google Authenticator or Microsoft Authenticator are much stronger. The code is generated on the device from a shared secret and the current time, it changes every 30 seconds, and it cannot be intercepted in transit because it never travels over the network. It can still be phished — a fake login page can ask for the code and relay it in real time — but that is much harder than intercepting an SMS.

**Hardware keys.** Security keys (like YubiKeys) are the strongest common second factor. The key checks the real domain before it signs anything, so a fake login page gets nothing — the key simply refuses to authenticate to a domain it does not recognise. This is called **phishing-resistant MFA**, and it is the standard that governments and large enterprises are moving toward. The downside is cost and logistics: every employee needs a key, keys get lost, and there has to be a process for replacing them.

**Biometrics.** Fingerprints and face recognition are convenient as a second factor, but they have a quirk: you cannot change your fingerprint if it leaks. They work best as a way to unlock a device or a key that does the actual authentication, rather than as the authentication itself.

### The right combination

So the answer is MFA, but specifically phishing-resistant MFA, combined with short-lived tokens after login. Passwords alone are too fragile for an enterprise, tokens alone just move the target, and MFA on its own is only as strong as its weakest factor.

A sensible enterprise setup looks like this:

1. **Password** (something you know) as the first factor, with proper hashing on the server, breach monitoring, and no forced rotation that pushes people toward predictable patterns.
2. **Phishing-resistant second factor** (something you have) — a hardware key or an authenticator app, not SMS.
3. **Short-lived tokens** after login, so the password is not sent on every request, and so a stolen token expires quickly.
4. **Risk-based checks** on top: new device, unusual location, or odd hour? Ask for the second factor again, even mid-session.
5. **A recovery process** that is as strong as the login itself — because attackers go for the recovery flow when the front door is locked.

## Comparison table

| | Password-based | Token-based | Multi-factor |
| --- | --- | --- | --- |
| Factors | One (know) | One (have, after login) | Two or more |
| Main strength | Simple, cheap, everyone understands it | Password not sent on every request; tokens expire and can be revoked | One stolen factor is not enough |
| Main weakness | Reuse, phishing, guessing, resets | Token theft; token is a new secret to protect | Weakest factor sets the strength; push fatigue |
| What breaks it | One leaked password | One stolen token | Two factors failing at once |
| Cost to run | Low | Medium (token infrastructure) | Medium to high (keys, apps, support) |
| User friction | Low (until a reset) | Low (invisible after login) | Higher (extra step every login) |
| Best for | Low-risk systems, as a first factor | APIs and sessions, after a strong login | Enterprise systems, admin accounts, anything sensitive |

## The trade-offs nobody can avoid

There is no authentication method with no downsides. Every choice trades security against usability, and usability matters — a system people work around is a system that is not secure, whatever the design says.

**MFA adds friction.** An extra step on every login is an extra step people resent, especially if it fails at the wrong moment. That is why enterprises increasingly use risk-based authentication: the second factor is required on a new device or an unusual login, but not every single time from a known laptop on the office network. The friction is spent where the risk is.

**Recovery is the weak spot.** The strongest MFA in the world is useless if the "I lost my second factor" process just asks for a password and an email. Attackers know this, and they target recovery flows and help desks. A recovery process needs to be at least as strong as the login it replaces.

**Tokens need care.** Short-lived tokens are safer, but they mean more refresh traffic and more complexity. Long-lived tokens are simpler, but a stolen one is useful for longer. There is no free choice here — only a choice about where to pay the cost.

**Hardware keys need logistics.** Buying, distributing, and replacing security keys for hundreds of people is a real operational job. Lost keys, broken keys, new starters, leavers — all of it needs a process. The security is better, but the organisation has to be ready to run it.

## Glossary

| Term | Meaning |
| --- | --- |
| Authentication | Proving who you are, usually with a password, token, or second factor. |
| Authorisation | What you are allowed to do once you are authenticated. |
| Factor | A kind of proof: something you know, have, are, or somewhere you are. |
| MFA | Multi-factor authentication — two or more factors combined. |
| Password hashing | Storing a one-way fingerprint of a password instead of the password itself. |
| Salt | A random value mixed into a password hash so identical passwords produce different hashes. |
| Credential stuffing | Trying leaked username/password pairs from one breach on many other sites. |
| Phishing | Tricking a user into handing over credentials on a fake login page. |
| SIM swap | Moving a victim's phone number to an attacker's SIM card to intercept SMS codes. |
| JWT | JSON Web Token — a signed token carrying identity and permissions. |
| Access token | A short-lived token used to access a system after login. |
| Refresh token | A longer-lived token used to get new access tokens without logging in again. |
| Token theft | Stealing a token and presenting it as if you were the legitimate user. |
| TOTP | Time-based one-time password — a code generated from a shared secret and the current time, changing every 30 seconds. |
| Phishing-resistant MFA | A second factor that cannot be tricked by a fake login page, such as a hardware security key. |
| Push fatigue / MFA fatigue | Flooding a user with approval requests until they approve one just to stop the notifications. |
| Risk-based authentication | Asking for extra proof only when something looks unusual, such as a new device or location. |

## Limitations

- This write-up compares the three approaches at a conceptual level. Real enterprise systems combine them — passwords plus tokens plus MFA plus risk checks — rather than picking one.
- The examples of breaches are described in general terms; the details of any specific incident are more complicated than a summary can show.
- Biometric authentication is covered briefly. It is a large topic on its own, especially around privacy, storage of biometric data, and what happens when biometric data leaks.
- Costs and logistics (buying hardware keys, running a help desk for resets) vary a lot between organisations, so the trade-offs here are described qualitatively, not with exact numbers.

## Further reading

- NIST SP 800-63B, Digital Identity Guidelines: Authentication and Lifecycle Management — the standard reference for authenticator types and their strength.
- OWASP Cheat Sheet Series: Authentication — practical guidance on passwords, sessions, and MFA.
- Microsoft / CISA guidance on phishing-resistant MFA — why hardware keys and FIDO2 matter.
- FIDO Alliance — the open standard behind passwordless and phishing-resistant authentication.

## The original post

The write-up above expands on the post below, which was written first. The post is reproduced here exactly as written.

> If you ask which authentication is most effective for an enterprise, the short answer is multi-factor. But I think the interesting part is why the other two keep failing, and why not all MFA is equal either.
>
> Password-based authentication is the weakest of the three, and it is not really the password's fault. A strong, unique password is fine on its own. The problem is people. In an enterprise with hundreds of employees, passwords get reused, written down, or phished. And once one password leaks in a breach, attackers try it everywhere. Hashing helps on the server side, but it does nothing if the user hands the password to a fake login page. Passwords also have a hidden cost, resets. Every reset is a support ticket and another chance for someone to social-engineer their way in.
>
> Token-based authentication, like JWTs or session tokens, fixes some of this. The user logs in once and the token does the rest, so the password is not flying around on every request. Tokens can expire, they can be revoked, and they can carry limited permissions. But a token is only as safe as where it is stored. If it is stolen from a browser or a device, the attacker does not need the password at all, they just present the token. So token-based auth moves the risk from the password to the token, it does not remove it.
>
> That is why MFA is the most effective for enterprise systems. It does not rely on one secret. Even if a password is phished, the attacker still needs the second factor, and even if a token is stolen, it usually expires fast and cannot be used to log in from scratch. The key point, though, is that the second factor matters a lot. An SMS code can be intercepted or SIM-swapped, and a push notification can be approved by a tired user at 2am, which is exactly how some real breaches happened. The stronger forms are authenticator apps and hardware keys, because they are much harder to phish. A hardware key checks the real domain before it signs anything, so a fake login page gets nothing.
>
> So my answer is MFA, but specifically phishing-resistant MFA, combined with short-lived tokens after login. Passwords alone are too fragile for an enterprise, tokens alone just move the target, and MFA on its own is only as strong as its weakest factor. The other thing enterprises get from MFA is a kind of safety net, when one layer fails, and one layer will fail eventually, the other one still holds. Does anyone else think the push-fatigue problem means we should stop using push approvals altogether, or is it just a training issue?

*Note: no classmates had posted yet when this was written, so no reply is included here.*
