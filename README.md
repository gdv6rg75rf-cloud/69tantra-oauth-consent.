# OAuth consent frontend

A small, generic, mobile-friendly sign-in and consent page for a staging Supabase OAuth server. It is not a backend or a document store.

## Public contents

- `index.html`: browser UI and client-side authentication logic.
- `.nojekyll`: serve the static site without a build step.
- This README.

Only the staging Supabase URL and publishable API key are included. They are public client configuration, not server credentials. Never add service-role keys, secret keys, personal access tokens, private repository links, workspace records, or documents here. Server authorization must remain deny-by-default and must not trust this page to grant access.

The UI is generic. There are no analytics, trackers, external fonts, or document APIs. The browser stores the authentication session under a dedicated storage key; GitHub does not receive the sign-in email through application code. The hosted page and source repository are public. A noindex directive is not access control and renaming a repository does not erase its prior public history.

## Publish

In repository Settings > Pages, choose:

- Source: Deploy from a branch
- Branch: main
- Folder: / (root)

Save. The site uses the repository root as the consent page; no nested route or app build is necessary.

## Staging integration prerequisites

Publishing the page does not complete OAuth setup. Before connecting a client:

1. Enable the staging Supabase OAuth server.
2. Set the Auth Site URL to the published GitHub Pages origin, and set the OAuth authorization path to `/oauth-consent/`.
3. Allow the exact consent page as an email redirect, including its authorization_id query. Do not enable an unrestricted redirect wildcard.
4. Provision the intended existing Auth user and server-side project grant. The page deliberately uses `shouldCreateUser: false`.
5. Register the OAuth client with its exact approved callback URI, or deliberately configure dynamic registration with consent.
6. Test the entire client -> sign-in -> consent -> code exchange -> scoped MCP request flow. Backend token validation, client authorization, and concurrent worker ownership require separate server tests.

Open email sign-in links in the same browser that began the flow, as this page uses PKCE. No server credential belongs in GitHub Pages.

## Dependency and verification

The browser SDK is pinned to `@supabase/supabase-js` 2.117.2. The script has SHA-384 Subresource Integrity, and the inline app script is covered by a CSP SHA-256 hash. After any inline-script edit, recalculate that CSP hash before publishing. No npm installation or paid build service is required.

Local syntax and 14 DOM/SDK-mock checks passed during preparation. They cover missing/duplicate requests, sign-in configuration, authenticated request display, approval, denial, redirect rejection, unavailable SDK, request mismatch, safe text rendering, prior-consent redirection, frame rejection, CSP consistency, and the public-content boundary. These are not evidence of a completed live OAuth connection.

Status: frontend source prepared; GitHub Pages publishing and live OAuth acceptance must be verified separately.

## Sign in with a six-digit email code (staging)

Configure the staging **Supabase Auth → Email Templates → Magic Link** template to send a six-digit OTP code instead of a clickable one-time link. Replace the link containing `{{ .ConfirmationURL }}` with the token `{{ .Token }}`, for example:

```html
<h2>OpenBrain sign-in code</h2>
<p>Enter this six-digit code on the OpenBrain staging authorization page:</p>
<p><strong>{{ .Token }}</strong></p>
```

The page uses Supabase `signInWithOtp` to send the email, then `verifyOtp({ email, token, type: 'email' })` to sign in. This avoids email-scanner prefetch consuming a single-use link. Codes are single-use and subject to email rate limiting. Publishing the frontend alone does not modify the Supabase email template. Do not change production templates.
