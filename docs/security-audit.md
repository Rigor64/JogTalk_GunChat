# Security Audit

Repository: `Rigor64/JogTalk_GunChat`
Date: `2026-05-08`

## Summary

This audit identified a hardcoded shared encryption key exposed in the repository source and documentation. The same value appears in application code and in the README, which means anyone with repository access can recover the key. Because the application is frontend/client-side, shipping a shared secret in the client bundle does not provide real secrecy.

## Findings

### 1. Hardcoded shared encryption key in application code

- **File:** `src/Chat.svelte`
- **Location:** around lines 47 and 69
- **Severity:** High
- **Type:** Hardcoded secret / shared encryption key

#### Evidence

The file contains a hardcoded key used to encrypt and decrypt chat messages:

```js
const key = '#rG.k4FALy-TB_gF';
const secret = await SEA.encrypt(newMessage, '#rG.k4FALy-TB_gF');
```

#### Why this is a problem

- Anyone who can read the source can recover the encryption key.
- If this value is bundled into frontend code, users can still extract it even if the repository becomes private.
- A shared static key means all users depend on the same secret, so compromise of that key compromises all protected messages.
- This weakens the claim of end-to-end encryption, because the secret is globally shared and distributed to every client.

#### Mitigation

- Remove the hardcoded shared key from source code.
- Do **not** use a global static secret for all users in client-side code.
- Prefer per-user or per-conversation keys derived through a secure key exchange mechanism.
- If this is only a demo, clearly label it as insecure and not suitable for production.
- Rotate any existing chat encryption material if old encrypted data is still considered sensitive.

#### Recommended remediation

- Refactor message encryption so the key is not a fixed literal in the client.
- If using GUN/SEA, design encryption around user-owned keys or room-specific keys distributed securely.
- Add a security note in the project docs describing the real security model.

---

### 2. Hardcoded shared encryption key documented in project documentation

- **File:** `README.md`
- **Location:** around line 133
- **Severity:** High
- **Type:** Secret disclosure in documentation

#### Evidence

The README explicitly publishes the master key:

```md
- The master key for the end-to-end encryption of the messages is: `rG.k4FALy-TB_gF`
```

#### Why this is a problem

- It publicly discloses the same key used by the application.
- It makes accidental misuse more likely by future contributors.
- It signals an insecure architecture choice and normalizes storing secrets in documentation.

#### Mitigation

- Remove the key from the README immediately.
- Replace it with a description of the intended encryption design, without including secrets.
- If the app uses a demo key for testing, reference it only in local developer instructions outside version control, or generate it locally at runtime for demos.

#### Recommended remediation

- Update the README to explain that production-safe encryption must not rely on a shared hardcoded client secret.
- Add a short “Security considerations” section documenting what is and is not protected.

---

## Additional files reviewed

The following files were reviewed during this audit and did not contain obvious API keys, private keys, or passwords in the inspected contents:

- `.gitignore`
- `package.json`
- `src/user.js`
- `src/Login.svelte`
- `.firebase/hosting.cHVibGlj.cache`
- `rollup.config.js`

## Notes

- The `.gitignore` excludes `.firebaserc` and `firebase.json`, which is good, but this does not guarantee secrets were never committed in repository history.
- No `.env` file was visible in the repository contents inspected.
- No private key material (PEM/SSH/OpenPGP) was found in the files reviewed.
- A deeper history scan is still recommended to verify that secrets were never committed and later removed.

## Next steps

1. Remove the key from `src/Chat.svelte`.
2. Remove the key from `README.md`.
3. Rotate any encryption material that depended on the exposed value.
4. Redesign encryption so secrets are not shared as hardcoded client literals.
5. Run a repository history scan for previously committed secrets.
6. Add a documented secret-handling policy for the project.
