# Vision

> The technical protocol lives in [SPEC.md](SPEC.md). This page explains *why* the project exists and where it could go. Nothing here is required to implement or use Habernero.

## The short version

Basic participation in modern life (logging in, paying, proving who you are, getting help) shouldn't depend on having a phone contract, an email account, or a monthly plan. Habernero is one small building block toward that: a way to approve things and prove control of an account using keys people hold themselves.

## 1. Identity without a phone number or email

Today, most services anchor your identity to a phone number or an email address. That works for many people, but it leaves others out or exposed:

- Phone numbers can be lost, reassigned, ported away by fraudsters (SIM swap), or simply unaffordable.
- Email and phone numbers are among the easiest ways to track a person across services.
- People who move, travel, are displaced, or are rebuilding their lives often lose access to both.

Habernero's goal: **no service using it should need your email address or phone number.** You hold a key, you approve requests over a private channel, and each service sees only a separate, unlinkable identifier.

## 2. Basic connectivity that doesn't require a monthly plan

If Wi-Fi is available, a cellular plan isn't strictly needed for a text message, an approval, or a login. Habernero is designed to work over any transport, including networks people run themselves:

- **Shared and community Wi-Fi.** Neighbors, libraries, schools and small businesses already share connectivity. A small, opt-in "basic services" lane (plain text messaging, approvals, and access to government and education sites) costs very little bandwidth.
- **Mesh networking.** Open router firmware such as [OpenWrt](https://openwrt.org) and mesh protocols such as [B.A.T.M.A.N.](https://www.open-mesh.org) and 802.11s let ordinary routers relay for one another, keeping essential messaging working during outages or in places with poor coverage.
- **Bluetooth and short-range links** for nearby devices, as a last-resort fallback for text and approvals.
- **Store-and-forward messaging.** Private messengers like SimpleX already tolerate intermittent connections, which suits community networks well.

This is about resilience and access: emergencies, rural areas, people between plans, and anywhere commercial coverage falls short.

## 3. Working with router makers, not around them

Custom firmware is powerful but intimidating. The aim is to make "share a safe, basic lane" a setting anyone can switch on:

- Work with router manufacturers on an opt-in **community/basic-access mode**: isolated from the owner's home network, rate-limited, limited to text and an allow-list of essential sites.
- Provide ready-made OpenWrt packages and setup guides for routers that already support open firmware.
- Respect the law and the owner's terms: sharing must be opt-in, isolated, and compatible with local regulations and internet service agreements.

## 4. Access to help

Being able to reach information and support, even without a paid plan, is part of the same picture. One related effort by this project's author:

- **AI legal information assistant** at **[ailegal.5pm.io](https://ailegal.5pm.io)** (part of 5pm.io / kwm59.io). It offers **general legal information and guidance, not legal representation**, and is not a substitute for a licensed lawyer. What it can offer, and what counts as legal advice, **varies by country and jurisdiction**.

## 5. Longer-term aspirations

These are ideas for the future, not current features. Each would need careful legal review and partners who share the goals.

- **Safe channels for reporting wrongdoing.** Privacy-preserving tools that help people report misconduct through lawful channels and connect with established civil-liberties and human-rights organizations, using the same no-phone, no-email identity.
- **A small community economy.** Lightweight, privacy-respecting payments (for example Monero, or other open systems) that let people contribute to and be paid for running community network nodes and relays, without subscriptions.
- **Open standards first.** Everything here should be documented openly, so anyone (communities, companies, public institutions) can implement it.

## Principles

1. **Humanist.** Built to widen access and protect people, especially those most easily left out.
2. **Lawful and cooperative.** Work with manufacturers, institutions and communities, within the law.
3. **Privacy by default.** Collect nothing that isn't needed; never require a phone number or email.
4. **Resilient.** Keep basic communication working when the usual infrastructure doesn't.
5. **Open.** Open specs, open source, no lock-in.
