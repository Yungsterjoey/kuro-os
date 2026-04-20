# KURO OS: Auth Flow

KURO OS supports three auth paths that converge on a single session
table. Every authenticated request carries `req.user.userId` (canonical
field, never `req.user.id`) once `requireAuth` middleware has run.

---

## 1. OTP (email)

```
Browser                    Express (server.cjs)             Mailer (nodemailer)
   │                             │                                  │
   │  POST /api/auth/otp/start   │                                  │
   │  { email }                  │                                  │
   ├────────────────────────────►│                                  │
   │                             │  generate 6-digit code           │
   │                             │  hash + store w/ TTL 10m         │
   │                             │  send code via SMTP              │
   │                             ├─────────────────────────────────►│
   │  202 Accepted               │                                  │
   │◄────────────────────────────┤                                  │
   │                             │                                  │
   │  POST /api/auth/otp/verify  │                                  │
   │  { email, code }            │                                  │
   ├────────────────────────────►│                                  │
   │                             │  compare hash, mint session      │
   │                             │  set httpOnly cookie             │
   │  200  Set-Cookie: kuro_sid  │                                  │
   │◄────────────────────────────┤                                  │
```

---

## 2. Google OAuth

```
Browser                    Express                          Google
   │  GET /api/auth/google/start │                                │
   ├────────────────────────────►│  302 → Google consent          │
   │◄────────────────────────────┤                                │
   │  …user authorises Google…   │                                │
   │                             │                                │
   │  GET /api/auth/google/cb    │                                │
   │  ?code=…                    │                                │
   ├────────────────────────────►│  exchange code for id_token    │
   │                             ├───────────────────────────────►│
   │                             │  verify with google-auth-lib   │
   │                             │  upsert user + mint session    │
   │  302 → /                    │                                │
   │  Set-Cookie: kuro_sid       │                                │
   │◄────────────────────────────┤                                │
```

---

## 3. Anonymous (L1 guest_gate)

```
Browser                    Express
   │  Any /api/* without cookie  │
   ├────────────────────────────►│
   │                             │  L1 guest_gate provisions ephemeral session
   │                             │  scope = guest, no write capability
   │  200 + Set-Cookie: kuro_gid │
   │◄────────────────────────────┤
   │                             │
   │  Subsequent requests carry  │
   │  the guest cookie until     │
   │  upgrade via OTP / OAuth.   │
```

Guest sessions can call public read endpoints (e.g.
`POST /api/neuro/phase/simulate`, `POST /api/pay/x402/parse-qr`) but are
refused on any route protected by `requireAuth`.

---

## Session storage

- Session rows live in SQLite (WAL).
- The `kuro_sid` cookie is `httpOnly`, `Secure`, `SameSite=Lax`.
- Every auth event is written to the L10 audit chain via
  `securityLog()`, including the negative path (failed OTP, OAuth
  refusal, replay attempt). Forensic replay reconstructs the auth
  history per session.
