# secrets-manager

A design and early C prototype for a self-hosted password and secrets manager: a small server that keeps an encrypted vault, plus browser-extension and CLI clients.

> Archived: no longer maintained. Kept for reference. The prototype is incomplete. Encryption and the network server are stubs, so it must not be used to store real secrets.

## What exists

The `server/` directory holds the start of `vault-server`, written in C (2022):

- Argument parsing for the `initialize`, `import`, `serve`, `help` and `version` commands
- Master-password and hardware-key prompts at startup
- Storage directory setup and validation
- CSV import scaffolding
- A small `Either` type for error handling
- Placeholders for encryption (`encryption.c`) and the TLS server (`commands/serve.c`)

See [server/manual.md](server/manual.md) for the intended command-line interface. Build with `make` inside `server/`.

## Intended design

### Server

- Starts only with a master password and a hardware key; the vault file is encrypted with keys derived from both
- Runs in an isolated Linux VM and serves clients over TLS
- Shuts itself down after 48 hours without a session, and after 30 days regardless
- Admin operations happen through the server executable, never over the network
- Pushes the encrypted vault to a backup server after every change; the backup API is idempotent and never deletes

### API

| Endpoint | Returns |
| --- | --- |
| `getSessionToken` | A session with a preset expiry |
| `getAccountByName` / `getAccountsByURL` | Username and password |
| `getAccount2FAByID` | Current TOTP code |
| `generatePassword` / `replacePasswordByID` | A new password, optionally saved |

### Clients

- **Firefox extension (desktop and mobile):** search by name, or list accounts for the current site, with copy buttons. Views clear themselves after 10 seconds.
- **CLI:** `vault get secret <ID>`, `vault get account.username <ID>`, `vault get account.password <ID>`

### Recovery

A "dead man's switch" recovery site was planned: trusted people could request access, the owner would be notified, and secrets would be released only after a waiting period and optional extra checks (multi-party approval, a phone call, proof of identity).

### Ideas not started

Tracking recovery emails and tags, and reports for duplicate passwords, accounts by email, and password age.

## Credits

The C examples in `server/examples/` and some socket and file-handling patterns follow [Beej's Guides](https://beej.us/guide/).
