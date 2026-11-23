# Habernero Protocol Specification

**Version:** 0.1 (draft, open for comment)
**Status:** Experimental. Not audited. Do not rely on it alone for high-value systems yet.

The key words MUST, SHOULD and MAY are used as described in RFC 2119.

---

## 1. Purpose

Habernero is an open, privacy-first way for a person to **approve something** (a login, a payment, a claim of ownership) by proving control of a key they already hold, over a private messaging channel, without handing the asking party a phone number, an email address, a password, or any balance history.

It is deliberately general. The same message format and the same verification rules cover:

| Use | Who asks (Relying Party) | What the person proves |
|---|---|---|
| Website / app login | The website | "I control the key registered to this account" |
| Card or payment fraud confirmation | Card issuer, bank, merchant, or payment processor | "I, the account holder, approve *this* amount to *this* payee, right now" |
| Proof of address ownership | Anyone (exchange, escrow, counterparty, forum) | "I control this address", **without** revealing balance or history |
| Generic action approval | Any service (admin action, withdrawal, API key creation) | "I approve this specific action" |

Habernero is **transport-agnostic** (SimpleX is the reference transport, but QR codes, deep links and others are allowed) and **key-agnostic** (Monero wallet signatures are the reference method, but Ed25519, Bitcoin BIP-322, Ethereum EIP-191 and passkeys can plug in).

## 2. Terms

- **Holder**: the person approving. Holds the key and runs a wallet or messenger.
- **Relying Party (RP)**: the service asking for approval (website, issuer, merchant, counterparty).
- **Challenge**: a short, human-readable text the RP creates and the Holder approves or signs.
- **Response**: what the Holder sends back: a tap result (Level 1) or a signature (Level 2).
- **Transport**: how the challenge and response travel (SimpleX, QR, deep link, copy/paste).
- **Method**: the signature scheme (e.g. `xmr`, `ed25519`, `btc-bip322`, `eth-eip191`, `webauthn`).
- **Binding**: the one-time enrollment step that links an RP account to a Holder's transport contact and/or public key.

## 3. Assurance levels

Habernero defines two levels. An RP picks the level per action, not per user.

| Level | Name | What it proves | UX | Use for |
|---|---|---|---|---|
| **L1** | Tap-approve | Control of the enrolled messaging contact (e.g. the SimpleX contact bound at enrollment) | One tap: `/approve` or `/deny` | Routine logins, small purchases, low-risk confirmations |
| **L2** | Signed approval | Control of the private key behind a public address | Sign the challenge in a wallet, return the signature | New device, password/key reset, large payments, withdrawals, proof of ownership |

**L1 is not key proof.** Anyone who has taken over the Holder's messenger profile can tap approve. RPs MUST NOT use L1 for account recovery, key rotation, or payouts.

An RP MAY require **L1 + L2** (approve in chat *and* sign) for its highest-risk actions.

## 4. The challenge

### 4.1 Canonical text format

The challenge is plain UTF-8 text so that a person can read it, and so it can be pasted into any wallet's "Sign message" box. Lines are `key: value`, separated by a single LF (`\n`), in **exactly** the order below. Optional fields that are absent are omitted entirely (no blank line). No trailing whitespace. No trailing newline.

```
Habernero approval request
<statement>

version: 1
domain: <rp domain>
purpose: <login | payment | ownership | action>
account: <rp account handle, opaque>
address: <holder public address or key id>
method: <xmr | ed25519 | btc-bip322 | eth-eip191 | webauthn>
amount: <decimal amount> <currency>
payee: <merchant or recipient display name>
payee-id: <merchant id, IBAN fragment, or address>
action: <short action code>
nonce: <base32, at least 128 bits>
issued-at: <RFC 3339 UTC>
expires-at: <RFC 3339 UTC>
request-id: <rp-chosen id>
```

### 4.2 Field rules

| Field | Required | Notes |
|---|---|---|
| line 1 `Habernero approval request` | yes | Fixed header. Signers and wallets can use it to warn "this is an authentication message, not a transaction". |
| `statement` | yes | One line, max 140 chars, written for a human: "Log in to example.com", "Approve 249.00 USD to ACME Store". Purely informational; the fields below are what is enforced. |
| `version` | yes | `1` for this spec. |
| `domain` | yes | The RP's registrable domain (e.g. `example.com`). The core anti-phishing field (§8.1). For non-web RPs, a stable identifier the Holder recognizes, published at enrollment. |
| `purpose` | yes | One of `login`, `payment`, `ownership`, `action`. A signature for one purpose MUST NOT be accepted for another. |
| `account` | yes for `login`/`action` | RP-side opaque handle. Not an email, not a real name. |
| `address` | yes for L2 | The public address/key the RP expects to sign. For Monero, the **subaddress** issued for this RP (§9.1). |
| `method` | yes for L2 | Signature scheme (§6). |
| `amount` | yes for `payment` | Decimal string and ISO 4217 or ticker code, e.g. `249.00 USD`, `0.75 XMR`. |
| `payee`, `payee-id` | yes for `payment` | What the Holder is paying. Prevents a signature being reused for a different merchant. |
| `action` | yes for `action` | Short code, e.g. `withdraw`, `api-key-create`, `change-email`. |
| `nonce` | yes | At least 128 bits from a CSPRNG, base32 (RFC 4648, no padding, lowercase). Single use. |
| `issued-at`, `expires-at` | yes | UTC. `expires-at - issued-at` SHOULD be ≤ 5 minutes for login and payment, ≤ 24 hours for ownership. |
| `request-id` | yes | Lets the RP find the pending request. |

### 4.3 Example: login

```
Habernero approval request
Log in to example.com from a new browser in Denver, US

version: 1
domain: example.com
purpose: login
account: u_7f3k2q
address: 8B...subaddress...Xy
method: xmr
nonce: 4w6qk3zj7m2hx5c9tqfa1v8yre
issued-at: 2026-11-23T00:05:00Z
expires-at: 2026-11-23T00:10:00Z
request-id: lg_01JD8Z
```

### 4.4 Example: card / payment confirmation

```
Habernero approval request
Approve 249.00 USD to ACME Store (card ending 4417)

version: 1
domain: issuer.example
purpose: payment
account: c_29dk1
address: 8C...subaddress...Qp
method: xmr
amount: 249.00 USD
payee: ACME Store
payee-id: mid_558201
nonce: ybq2r7k1x0d4m9svh3ce6tnw5a
issued-at: 2026-11-23T00:05:00Z
expires-at: 2026-11-23T00:08:00Z
request-id: tx_88123
```

### 4.5 Example: proof of address ownership

```
Habernero approval request
Prove control of this address to escrow.example

version: 1
domain: escrow.example
purpose: ownership
address: 84...address...Zt
method: xmr
nonce: k8w2n5d1q7c3vx9mfz4hr6yt0e
issued-at: 2026-11-23T00:05:00Z
expires-at: 2026-11-24T00:05:00Z
request-id: own_4410
```

This proves key control **at the moment of signing**. It says nothing about balance and reveals no transaction history (§9.3).

### 4.6 Machine envelope (optional)

When a transport can carry structured data, the RP MAY also send a JSON envelope. The canonical text in `challenge` is what gets signed; the other fields are hints and MUST be checked against it.

```json
{
  "hab": 1,
  "challenge": "Habernero approval request\nLog in to example.com ...",
  "callback": "https://example.com/.well-known/habernero/respond",
  "levels": ["L1", "L2"],
  "rp_sig": "<optional: RP's Ed25519 signature over challenge>"
}
```

`rp_sig` lets a Holder's client verify that the challenge really came from the RP's published key (§7.3), which defeats spoofed requests arriving over a channel the attacker controls.

## 5. Message flows

### 5.1 Enrollment (binding)

Done once per RP account, while the Holder is already authenticated some other way (or creating the account).

1. RP shows a SimpleX contact address or one-time invitation link/QR for its Habernero bot.
2. Holder connects from their messenger. RP's bot sends a code; Holder types it into the RP's web page (or the page shows a code the Holder sends to the bot). This links **this SimpleX contact** to **this RP account**.
3. For L2, the Holder gives the RP a public address. With Monero, the Holder SHOULD create a **new subaddress dedicated to this RP** (§9.1).
4. RP sends an `ownership` challenge for that address. Holder signs it. RP verifies (§6) and stores `(account, method, address)`.
5. RP stores the SimpleX contact ID. It never needs a phone number or email.

### 5.2 Level 1: tap-approve (SimpleX reference transport)

```
RP server                 RP bot (simplex-chat CLI)          Holder's SimpleX app
    |  create challenge          |                                   |
    |--------------------------->|  message: statement + summary     |
    |                            |  + "/approve <request-id>"        |
    |                            |  + "/deny <request-id>"           |
    |                            |---------------------------------->|
    |                            |                     tap /approve  |
    |                            |<----------------------------------|
    |  check contact, request-id,|                                   |
    |  expiry, not used          |                                   |
    |<---------------------------|                                   |
    |  mark used, allow action   |                                   |
```

Implementation note: the reference bot drives the `simplex-chat` CLI through its local WebSocket API (`simplex-chat -p <port>`). SimpleX renders `/command` text as tappable in chat. The bot MUST accept an approval only from the contact bound to the account in §5.1, and only for a `request-id` that is pending, unexpired and unused.

`/deny` MUST cancel the request and SHOULD be treated as a fraud signal for payments (e.g. freeze the card pending review).

### 5.3 Level 2: signed approval

1. RP builds the canonical challenge (§4) and stores it with status `pending`.
2. RP delivers it over any transport:
   - **SimpleX**: bot sends the text in a code block (easy to copy) plus a link to a web page with a copy button.
   - **QR / deep link**: a page shows the text and a QR for wallets that can read it.
   - **Copy/paste**: the web page shows the text with a copy button.
3. Holder opens their wallet, chooses "Sign message", **selects the address shown in `address`**, pastes the challenge, signs.
4. Holder returns the signature by replying to the bot, pasting into the page, or posting to `callback`.
5. RP verifies (§6) and marks the challenge `used`.

A future companion app or wallet plug-in MAY automate steps 3 and 4 (read challenge, show a clear confirmation screen, sign, send). The protocol does not change.

### 5.4 Payment confirmation flow

1. Merchant or issuer risk engine flags a transaction (or policy says "always confirm").
2. Issuer creates a `payment` challenge containing the exact amount and payee.
3. Below the issuer's threshold: L1 tap. Above it, or for new payees: L2.
4. Approve → authorize. Deny or timeout → decline and alert.

Where card networks already require EMV 3-D Secure, Habernero is intended to act as the issuer's **out-of-band authenticator** inside the 3DS challenge step, not a replacement for 3DS. That integration is up to each issuer.

## 6. Verification

### 6.1 Steps an RP MUST perform (all levels)

1. Look up `request-id`. Reject if not found, not `pending`, or already used.
2. Reject if the current time is after `expires-at` (allow at most 30 s clock skew).
3. Mark the request `used` **atomically** before (or in the same transaction as) allowing the action, so two racing responses cannot both succeed.
4. For L1: confirm the reply came from the bound transport contact.
5. For L2: rebuild the canonical text **from the RP's own stored copy**, never from what the Holder sends back, then verify the signature against the stored `address` with the stored `method`.
6. Confirm `purpose`, `domain`, `amount`, `payee` in the stored copy match the action being authorized.
7. Log the outcome (request-id, level, result, time). Do not log the full signature next to identifying data longer than needed.

### 6.2 Method: `xmr` (Monero message signature, reference)

- Holder signs with the Monero GUI/CLI/Feather "Sign message" function, using the spend key (the default) for the selected (sub)address. Signatures begin with `SigV2`.
- RP verifies with `monero-wallet-rpc`:

```json
POST /json_rpc
{ "jsonrpc": "2.0", "id": "0", "method": "verify",
  "params": { "data": "<canonical challenge>",
              "address": "<stored address>",
              "signature": "SigV2..." } }
```

Result `{"good": true}` (plus signature type on newer versions) means the signature is valid. The verifying `monero-wallet-rpc` can run with any throwaway wallet open and **does not need a daemon connection or any of the Holder's keys**. Run it bound to localhost only.

- RP MUST reject signatures made with the **view** key (`signature_type: "view"` on wallet versions that report it). A view-key signature only proves someone can see incoming funds, not that they can spend.

### 6.3 Other methods (pluggable)

| Method | Signer | Verify with |
|---|---|---|
| `ed25519` | Any Ed25519 key (e.g. a companion app key) | libsodium `crypto_sign_verify_detached` over the canonical bytes |
| `btc-bip322` | Bitcoin wallets supporting BIP-322 | A BIP-322 verifier library |
| `eth-eip191` | Ethereum wallets (`personal_sign`) | `ecrecover`; compare to stored address |
| `webauthn` | Passkeys / hardware keys | Standard WebAuthn assertion; `challenge` = SHA-256 of canonical text |

New methods are added by registering a name and a verification procedure; the challenge format does not change.

## 7. Transports

### 7.1 SimpleX (reference)

Why: no phone number or user ID, per-contact queues, end-to-end encryption, self-hostable relays. The RP runs `simplex-chat` as a bot with its own profile.

Rules:
- One bound contact per RP account (more MAY be allowed for multiple devices).
- The bot MUST show the **domain** and **statement** in every message, so the Holder sees who is asking.
- The bot SHOULD warn: "Never sign a Habernero request you did not start" for login.

### 7.2 Other transports

QR code on a web page, `habernero:` deep link carrying the JSON envelope, email (weak, for fallback only), or manual copy/paste. Transport choice does not change what is signed.

### 7.3 RP signing key (recommended)

RPs SHOULD publish an Ed25519 public key at `https://<domain>/.well-known/habernero.json`:

```json
{ "hab": 1, "rp_keys": ["ed25519:<base64 pubkey>"], "bot": "simplex:/contact#..." }
```

Clients that support it verify `rp_sig` before showing a request, and show a hard warning if it fails.

## 8. Threat model

| Threat | What happens | Mitigation |
|---|---|---|
| **Phishing / fake site** | Attacker site asks the Holder to sign a challenge it got from the real RP | The signed text contains `domain`. Wallet/companion UIs SHOULD show it prominently; RPs reject any challenge they did not issue. Remaining risk: a Holder who signs without reading. Best defense is a companion app that checks `domain` against the page the user is on and `rp_sig`. |
| **Real-time relay (MITM)** | Attacker proxies the real login and forwards the real challenge to the victim | Short expiry, `statement` that names the device/location, and L1 messages coming from the RP's bound bot (an attacker can't send *as* that contact). This is the same residual risk as push-2FA; it is reduced, not eliminated. |
| **Replay** | Old signature reused | Single-use `nonce` + `request-id`, atomic `used` flag, expiry. |
| **Cross-purpose / cross-RP reuse** | Login signature used to approve a payment, or at another site | `purpose` and `domain` are inside the signed text; per-RP subaddresses. |
| **Amount / payee swap** | Malware changes what the user thinks they approve | Amount and payee are in the signed text; RP verifies against its own stored copy. Holder must read the wallet's sign screen (WYSIWYS). |
| **SIM swap** | Attacker takes over phone number | Not applicable: no phone number is used. |
| **Messenger account takeover** | Attacker controls the Holder's SimpleX profile | Gains L1 only. L2 still requires the wallet key. Keep recovery and payouts at L2. |
| **Push fatigue / spam approvals** | Attacker floods approval requests hoping for a tap | Rate-limit requests per account; show context; after 3 denials, lock and require L2. |
| **Key compromise** | Wallet seed stolen | Attacker can pass L2. Holder rotates: enroll a new subaddress from a fresh wallet via an L2 approval from the old key (if not yet abused) or RP's out-of-band recovery. Recommend a dedicated "auth wallet" with no funds. |
| **RP database leak** | Stored data exposed | RP stores only an opaque account handle, a SimpleX contact ID, and a per-RP public subaddress. No password hash, phone, or email required. |
| **Malicious RP** | RP tries to track the Holder across sites | Per-RP subaddresses and per-RP SimpleX contacts (§9). |
| **Blind signing a transaction** | Challenge text crafted to look like something else | Fixed header line; methods here sign messages, not transactions. Monero message signatures cannot move funds. |

Out of scope: a fully compromised Holder device (it can sign anything), and coercion.

## 9. Privacy

### 9.1 One subaddress per Relying Party

A Monero wallet can create unlimited subaddresses that cannot be linked to each other or to the main address by outsiders. Holders SHOULD give each RP its own fresh subaddress. Two RPs comparing notes then see two unrelated addresses. RP software MUST NOT ask for the primary address specifically.

Other methods: use per-RP derived keys (e.g. HD derivation path per domain, or one passkey per site, which WebAuthn already does).

### 9.2 View keys are never shared

Habernero never asks for a view key, a spend key, or a seed. An RP that requests a view key "to verify ownership" is not following this spec. Signing proves control; a view key would expose incoming transaction history.

### 9.3 Ownership without balances

A Habernero `ownership` signature proves "I control this address now". It reveals:
- no balance,
- no transaction history,
- no link to other addresses.

This answers "who are you / do you control this?" with fresh proof instead of a bank statement or balance snapshot that is stale the moment it's printed.

If an RP genuinely needs **proof of funds**, that is a different, more revealing step (e.g. Monero `get_reserve_proof`, which discloses an amount of at least X). It is outside Habernero v1 and MUST be a separate, explicit request, never bundled into login.

### 9.4 Transport privacy

SimpleX has no global user identifiers; each RP gets a separate contact and queue, so RPs cannot correlate the Holder by messenger identity either. RPs SHOULD let Holders use their own SimpleX relays.

### 9.5 Data minimization for RPs

Store: account handle, method, public (sub)address, transport contact ID, request log for the retention period your regulator requires. Don't store: IP-to-address mappings beyond fraud needs, full challenge texts after expiry (keep a hash).

## 10. Conformance checklist

An RP implementation conforms to v1 if it:

- [ ] Builds challenges in the exact canonical format (§4.1) with ≥128-bit nonces.
- [ ] Verifies only against its own stored challenge copy (§6.1 step 5).
- [ ] Enforces single use atomically and enforces expiry.
- [ ] Rejects cross-purpose use and view-key signatures.
- [ ] Never uses L1 alone for recovery, key change, or payouts.
- [ ] Never asks for a view key, spend key, seed, phone number or email as part of Habernero.

## 11. Possible extension: signed printable check / payment authorization

Idea: when someone insists on "a real check", the Holder generates a printable PDF check (or a written payment authorization) and Habernero adds a verifiable approval to it.

How it would fit:
- The payer creates a `payment` challenge for the check's exact amount, payee, date and check number, and signs it (L2).
- The PDF carries a QR code containing the canonical challenge and signature. The payee (or their bank) scans it and verifies that the account holder really approved *this* check for *this* amount to *this* payee. Altered amounts or payees fail verification.

Real-world limits (US):
- Printing your own checks is legal, but the bottom line must be a correct **MICR E-13B** line (routing, account, check number). Many banks and check scanners still expect MICR toner/ink and proper check stock; plain inkjet output may be rejected or need manual handling.
- Whether a bank accepts a self-printed check is the bank's call. Remote deposit usually works from an image; teller and lockbox processing varies.
- A paper check exposes your routing and account numbers to everyone who handles it, which is the opposite of Habernero's privacy goal and a common source of check fraud. Positive Pay (if your bank offers it) is the stronger protection.
- Banks don't read the QR today, so the signature protects the payee and helps disputes; it doesn't stop the bank from paying an altered check.

Treat this as an optional add-on, not part of v1 conformance.

## 12. Open questions (for the GitHub discussion)

1. Should the canonical format be JSON instead of text? (Text was chosen so any wallet's existing "Sign message" box works today.)
2. A `habernero:` URI scheme and a small companion app to remove copy/paste.
3. Multi-device: several bound SimpleX contacts per account, or one?
4. Threshold approvals (2-of-3 holders) for shared accounts.
5. Formal mapping to EMV 3DS out-of-band authentication for issuers.
6. Test vectors: fixed challenges with known Monero/Ed25519 signatures.

## 13. Prior art and related work

- **Sign-In with Ethereum (EIP-4361 / SIWE)**: the closest model for the human-readable signed login message with domain, nonce and expiry. Habernero borrows that shape but is chain-agnostic, adds payment and ownership purposes, and adds a private push transport.
- **MoneroAuth** (`github.com/MoneroAuth/web-demo`): an existing demo of website login with Monero signatures. Worth reviewing and crediting; Habernero differs by adding SimpleX push approval, tiered levels, payment binding and per-RP subaddress privacy rules.
- **django-cryptolock** (`github.com/dethos/django-cryptolock`): Django auth backend using Monero/Bitcoin signed challenges.
- **WebAuthn / passkeys**: strong phishing-resistant login built into browsers; Habernero can carry WebAuthn as a method.
- **Push-2FA apps** (Duo, bank apps): the L1 tap model, but tied to phone numbers or vendor accounts.
