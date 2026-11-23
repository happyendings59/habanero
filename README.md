# Habernero 🌶️

**Private approvals for logins, payments and proof of ownership. No phone number, no password, no balance snapshots.**

Habernero is an open protocol. A service (a website, card issuer, merchant or counterparty) sends you a short approval request over a private messenger like [SimpleX](https://simplex.chat). You either:

- **tap `/approve`** for routine things (Level 1), or
- **sign the request with your wallet key** for sensitive things (Level 2): a new device, a big payment, a withdrawal, or proving you control an address.

The signed text names the site, the purpose, and (for payments) the exact amount and payee, so the signature can't be reused somewhere else or for a different amount.

## Why

| Today | With Habernero |
|---|---|
| SMS codes: SIM swap, phone number tied to your identity | No phone number. SimpleX has no user IDs at all |
| Passwords + email resets | A key you already hold signs a one-time challenge |
| Card fraud texts that look exactly like phishing | Request is bound to the issuer's domain and the exact amount and payee; deny freezes it |
| "Send us a bank statement / balance screenshot" | Fresh signature proves control right now, with no balance or history revealed |
| Same identity reused everywhere | One Monero subaddress and one SimpleX contact per service, so services can't link you |

## How it works (short version)

1. **Enroll once:** connect to the service's SimpleX bot and give it a fresh subaddress made just for that service. Sign one challenge to prove it's yours.
2. **Login or payment:** the service's bot messages you: *"Approve 249.00 USD to ACME Store?"* `/approve` `/deny`.
3. **Sensitive action:** the bot sends a text challenge. Paste it into your wallet's **Sign message**, send the signature back. The server checks it with `monero-wallet-rpc verify`.

Full details: **[SPEC.md](SPEC.md)**. Why we're building it: **[VISION.md](VISION.md)**.

## Works with more than Monero

The message format is key-agnostic and transport-agnostic.

- **Keys:** Monero (reference), Ed25519, Bitcoin BIP-322, Ethereum EIP-191, passkeys/WebAuthn.
- **Transports:** SimpleX (reference), QR code, deep link, copy/paste.

## Status

Draft v0.1 spec. No reference implementation yet. Planned:

- [ ] Reference SimpleX bot (uses the `simplex-chat` CLI WebSocket API)
- [ ] Verifier library (Monero via `monero-wallet-rpc`, Ed25519)
- [ ] Test vectors
- [ ] Demo login page
- [ ] Companion app to remove copy/paste

## Not a security guarantee

This is unaudited. Level 1 (tap) only proves someone controls your messenger, not your key. Never sign a request you didn't start. Never give anyone your seed, spend key or view key: Habernero never needs them.

## Related projects

- [Sign-In with Ethereum (EIP-4361)](https://ethereum.org/en/developers/docs/ethereum-stack/authentication/): the signed-message login pattern this borrows from
- [MoneroAuth web demo](https://github.com/MoneroAuth/web-demo): earlier Monero-signature login demo
- [django-cryptolock](https://github.com/dethos/django-cryptolock): Django login with Monero/Bitcoin signatures

## License

This repository is licensed under **Apache-2.0** (see [LICENSE](LICENSE)), which includes a patent grant, which matters for an auth protocol. The spec text may later be dual-licensed under CC BY 4.0 to make it easier to quote.

## Contributing

Open an issue or discussion. Especially wanted: wallet developers, SimpleX bot developers, and anyone from card issuing or payments.
