# Secure setup for GitHub Pages + Google Sheets backend

This app can run on GitHub Pages securely **if Google Sheets access is protected by server-side identity checks**.

## Recommended architecture

Use this flow:

1. Host `index.html` on GitHub Pages.
2. Add **Google Sign-In (Google Identity Services)** to the page.
3. Send the returned **ID token** with each API request to your Apps Script Web App.
4. In Apps Script, verify the ID token with Google and allow only approved users.
5. Only after verification, read/write Google Sheets.

This keeps authorization on the server side (Apps Script) and avoids trusting client-side PIN checks.

## Why this is secure (for this stack)

- GitHub Pages is static, so secrets cannot be stored safely in the browser.
- ID token verification happens server-side.
- Access can be restricted to a fixed email allowlist (for example, owner + imam).
- Roles (owner/imam/viewer) can be enforced in Apps Script before writes.

## Apps Script requirements

Implement these controls in the Web App:

- Verify `idToken` with `https://oauth2.googleapis.com/tokeninfo?id_token=...`.
- Validate at least:
  - `aud` matches your Google OAuth Client ID
  - token not expired
  - email is verified
- Enforce an allowlist (Script Properties or a protected sheet tab).
- Enforce role-based permissions:
  - viewers: read only
  - imam/owner: write attendance
  - owner: manage points/PIN-equivalent settings
- Reject requests without valid token.
- Log writes (timestamp, actor email, action, key).

## Frontend requirements

- Replace client-side PIN-only trust with Google sign-in state.
- Include ID token in every `get/set/list` request payload.
- Keep auth state in `sessionStorage` only (short-lived UX convenience).
- Handle `401/403` by signing users out and showing a clear error.

## Deployment hardening

- Set Apps Script Web App access to only what your sign-in model needs.
- Check request origin and only allow your GitHub Pages origin.
- Add simple abuse protection (rate limits per user/IP where possible).
- Never embed private keys or long-lived secrets in `index.html`.

## Migration checklist from current version

- [ ] Add Google Identity Services sign-in to frontend.
- [ ] Update API calls to send `idToken`.
- [ ] Implement token verification + allowlist in Apps Script.
- [ ] Move role checks fully to Apps Script.
- [ ] Remove default/fallback PIN behavior for privileged actions.
- [ ] Test with authorized and unauthorized accounts.

## Minimum viable secure baseline

If you want the smallest secure upgrade while staying on GitHub Pages + Sheets:

- Keep the static frontend.
- Add Google Sign-In.
- Enforce all authorization in Apps Script using verified ID tokens.

That is the most practical secure approach for this repository's deployment model.
