# Vaultity

A **serverless, privacy-first password manager** that users can deploy themselves. No centralized backend, no vendor lock-in, just **end-to-end encryption (E2EE)**, **simplicity**, and **portability**.

---

## 🌍 Vision

* **Privacy-first**: Only encrypted blobs leave the client.
* **Serverless**: Deploy on free-tier providers like **Vercel** or **Cloudflare Workers**.
* **Self-hostable**: No dependency on a centralized service.
* **Portable**: Works across devices with biometric unlock support.

---

## 🏗 Architecture

### Data Storage

* **MongoDB Atlas (Free tier)** used for storing vault blobs.
* Only **encrypted blobs** are stored; no plaintext credentials or raw keys.

### Encryption Model

* Vault protected with a **randomly generated encryption key (vault key)**.
* Vault key is wrapped twice:

  * 🔑 With **user’s master password**.
  * 🔑 With **server master password**.
* When user changes password → only the wrapping is updated.

### Biometric Unlock

* Biometric systems (FaceID, TouchID, Windows Hello, Android Keystore, etc.) unlock the **wrapped vault key**.
* The raw encryption key never leaves secure hardware storage.

---

## 🗂 System Design

### High-Level Flow

```
                  ┌───────────────────────┐
                  │       Client App       │
                  │  (Web / Android / Ext) │
                  └───────────┬───────────┘
                              │
                              ▼
        ┌───────────────────────────────┐
        │   Vault Key (randomly gen.)   │
        └───────┬───────────┬──────────┘
                │           │
   ┌────────────┘           └───────────────┐
   ▼                                        ▼
Encrypted with User PW              Encrypted with Server PW
(AES-GCM + Argon2)                  (AES-GCM + KDF)
   │                                        │
   ▼                                        ▼
┌───────────────────────┐        ┌───────────────────────┐
│  Wrapped Vault Key U  │        │  Wrapped Vault Key S  │
└───────────┬───────────┘        └───────────┬───────────┘
            │                                 │
            └──────────────┬──────────────────┘
                           ▼
                 Stored in MongoDB Atlas
                 (only encrypted blobs)

   ┌─────────────────────────────────────────────┐
   │ Credential Data (titles, usernames, pwds)   │
   │ All encrypted with Vault Key (AES-GCM)      │
   └─────────────────────────────────────────────┘

```

### Vault Lifecycle

1. **Vault Creation**

   * Client generates random **vault key**.
   * Vault key encrypted twice → with user master password & server password.
   * Encrypted vault + metadata stored in MongoDB.

2. **Adding Credentials**

   * Client encrypts credentials with vault key (AES-GCM).
   * Uploads ciphertext blob to MongoDB via serverless API.

3. **Unlocking Vault**

   * User authenticates (password or biometrics).
   * Wrapped key is decrypted locally.
   * Vault key unlocked → credentials decrypted in memory only.

4. **Password Change**

   * Vault key stays the same.
   * Only the wrapped user-key is re-encrypted with the new password.

---

## ✅ Roadmap / Checklist

### Phase 1 – Foundations

* [ ] Define tech stack (Next.js / React frontend + Vercel/Cloudflare Workers backend).
* [ ] Initialize Git repo & project structure.
* [ ] Set up environment configuration system.
* [ ] Implement serverless API routes.
* [ ] Connect to MongoDB Atlas (free tier).

### Phase 2 – Core Security

* [ ] Implement vault creation with random encryption key.
* [ ] Implement AES-GCM encryption for vault contents.
* [ ] Implement key wrapping:

  * [ ] Encrypt vault key with user master password (PBKDF2/Argon2 + AES).
  * [ ] Encrypt vault key with server master password.
* [ ] Implement password change → rewrap only keys.

### Phase 3 – Client Features

* [ ] Login & vault unlock with password.
* [ ] Save credential (title, username, password, notes).
* [ ] Edit / delete credential.
* [ ] Autofill integration.

### Phase 4 – Biometric Integration

* [ ] iOS/macOS → Secure Enclave integration.
* [ ] Android → Keystore integration.
* [ ] Windows → Windows Hello integration.
* [ ] WebAuthn → FIDO2 biometric unlock fallback.

### Phase 5 – Apps & Extensions

* [ ] Android native app.
* [ ] Browser extension (Chrome/Firefox/Edge).

### Phase 6 – UX & Portability

* [ ] Responsive UI for desktop & mobile.
* [ ] Import/export encrypted vaults.
* [ ] Offline-first sync model.
* [ ] Add dark/light themes.

### Phase 7 – Deployment & Docs

* [ ] One-click deploy to Vercel/Cloudflare Workers.
* [ ] Setup CI/CD with GitHub Actions.
* [ ] Write developer documentation.
* [ ] Publish user guide.

---

## 🔒 Security Considerations

* Use **Argon2id** or **scrypt** for password-based key derivation.
* Always encrypt with **AES-GCM (authenticated encryption)**.
* Enforce **zero-knowledge**: servers must never see raw keys.
* Consider **end-to-end testing with threat modeling**.
* Keep server master password rotated regularly.

---

## 📌 Future Ideas

* [ ] Support passkeys (FIDO2/WebAuthn) as alternative to passwords.
* [ ] Add organizations / shared vaults.
* [ ] Build iOS app.
* [ ] Desktop client (Electron or Tauri).

---

## 📜 License

MIT (to keep it free & community-driven).
