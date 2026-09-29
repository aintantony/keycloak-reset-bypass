# CVE-2026-18963 — Keycloak credential-reset authentication bypass

A proof-of-concept for an authentication bypass in Keycloak's
`reset-credentials` flow. An attacker can reset an arbitrary user's
password **without reading the reset email** and then log in as that user.

## What it is

Keycloak's reset-credentials flow lets an unauthenticated visitor submit a
username and, under certain conditions, reach the password-update form
without ever presenting a valid action token. This PoC walks that flow and
sets a new password for a chosen account.

## Affected version

- **Vulnerable:** Keycloak < 26.7.2
- **Fixed:** 26.7.2

If the target serves the update-password form without an action token, it is
vulnerable. On a patched build the flow stops early and the script reports
`target is likely patched`.

Requirements: Keycloak with the *Forgot password* flow enabled
(`reset-credentials`).

## Usage

```bash
./exploit.sh <base-url> <username> <new-password> [--realm REALM] [--client-id ID] [--proxy URL]
```

Example:

```bash
./exploit.sh http://127.0.0.1:9999/keycloak koko qwerqwer
```

Options:

| Option | Default | Description |
|--------|---------|-------------|
| `url` | — | Keycloak base URL, e.g. `http://host/keycloak` |
| `victim` | — | Username to take over |
| `password` | — | New password to set |
| `--realm` | `cve-lab` | Realm name |
| `--client-id` | `account` | OIDC client used to enter the login flow |
| `--proxy` | — | HTTP proxy, e.g. `http://127.0.0.1:8888` |

**Note:** The `--realm`, `--client-id`, and `--proxy` parameters are optional. If not specified, the script uses the default values shown above.

On success it prints the vulnerable result and exits `0`; otherwise it
explains where the flow stopped and exits `1`.

Requires `bash`, `curl`, `grep -E/-o/-F` and `sed`.

## Migration

**Upgrade to Keycloak 26.7.2 or later.** That release restores the action-token
requirement in the reset-credentials flow.

If you cannot upgrade immediately:

- Disable the *Forgot password* / `reset-credentials` flow in the affected
  realms until patched.
- Behind a reverse proxy, rate-limit and log `/realms/*/login-actions/*` on
  `POST` to catch the bypass attempts.
- Rotate credentials for any account that may have been reset during the
  exposure window; the reset leaves no mail trail on the victim's side.
