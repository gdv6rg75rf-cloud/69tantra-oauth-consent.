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

## Password recovery without the unpublished GitHub Pages root

The staging Supabase Auth project's `Site URL` currently resolves password recovery to `https://gdv6rg75rf-cloud.github.io/`, which has no published GitHub Pages site. Do not change the project-wide Site URL for this fix because the OAuth authorization path uses it.

Instead, start recovery **from an OAuth authorization link** that opens `https://gdv6rg75rf-cloud.github.io/oauth-consent/?authorization_id=...`. The login form's **Forgot password? Send recovery email** button calls `resetPasswordForEmail` with the exact redirect:

`https://gdv6rg75rf-cloud.github.io/oauth-consent/?flow=password-recovery`

Ensure this exact redirect URL is allow-listed in staging Supabase **Authentication → URL Configuration → Redirect URLs**. The recovery page uses the existing PKCE-capable Supabase client to establish a session, shows a new-password form, and updates the existing signed-in user via `auth.updateUser({ password })`.

Avoid using the Supabase Dashboard's **Send password recovery** action while the Site URL points to the unpublished root: that action does not supply this per-request redirect. Email security scanners may consume single-use recovery links before they reach the user; this change fixes the 404 and missing reset form, not prefetch behavior. For that separate problem, use code-based OTP after the staging SMTP/template setup is available.

This frontend change does not alter Auth users, grants, project-wide URLs, or production.
