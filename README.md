# atvero-onboarding

Self-service onboarding page for customers enrolling in Atvero SPFx
auto-deploys. A SharePoint administrator signs in, the page uses their
delegated Microsoft Graph token to grant the **Atvero Deployer** Entra app
`FullControl` on a single SharePoint site (their app catalog), and
displays the three bits Atvero needs to add them to the customer registry.

No backend. No secrets. The admin's token never leaves their browser —
it makes one Graph call and is discarded.

## How it works

1. Atvero emails the customer admin a link:
   `https://atvero.github.io/atvero-onboarding/?site=https://contoso.sharepoint.com/sites/apps`
2. Admin clicks **Sign in and grant access**, signs in via MSAL.
3. Page calls:
   - `GET /sites/{hostname}:/{path}` to resolve siteId.
   - `POST /sites/{siteId}/permissions` with `roles: ["fullControl"]` and
     the Atvero Deployer app identity.
4. Admin is shown tenant ID, domain, and app catalog URL, plus a
   pre-filled mailto: link back to Atvero's onboarding inbox.

## Setup

Before publishing, replace the placeholders in `index.html`:

| Placeholder | Replace with |
|-------------|--------------|
| `REPLACE_WITH_PIM_APP_CLIENT_ID` | Client ID of the existing Atvero PIM Entra app (the one with delegated `Sites.FullControl.All` already admin-consented by customers). |
| `REPLACE_WITH_DEPLOYER_APP_ID` | Client ID of the **Atvero Deployer** app — the app that will be granted site-scoped access. |
| `onboarding@atvero.com` | The inbox customers should mail their tenant details to. |

### Entra app reg — one-time configuration

The page signs in against the existing PIM app registration. It needs:

1. **Single-page application** platform configured with this page's URL as
   a redirect URI (e.g. `https://atvero.github.io/atvero-onboarding/`).
   App registrations → your PIM app → Authentication → Add a platform →
   Single-page application.
2. Delegated Microsoft Graph **`Sites.FullControl.All`** — already present.
3. Admin-consented — already present for existing customers. New customers
   will see a consent prompt the first time they use the onboarding page.

No changes to the Atvero Deployer app are needed here.

## Deploy

GitHub Pages from `main`, static HTML, no build. Enable Pages on the repo,
root directory, `main` branch.

For a custom domain (`onboard.atvero.com`), add a `CNAME` file with the
hostname and configure DNS.

## Revocation

See Atvero Deployer onboarding docs. TL;DR:
`Revoke-PnPAzureADAppSitePermission -Site <url> -PermissionId <id>`
or remove the permission via SharePoint admin.
