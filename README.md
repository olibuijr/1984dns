# 1984dns

A command-line tool for managing DNS zones on [1984.is](https://1984.is) — the Icelandic privacy-focused hosting provider.

One-liner DNS record updates from your terminal. No web UI needed.

## Features

- **List, create, update, and delete** DNS records across all your 1984.is zones
- **Auto-discovery** of zones on your account (cached locally)
- **Upsert logic** — `set` updates existing records or creates new ones
- **Session caching** — logs in once, reuses the session until it expires
- **Shorthand commands** — `point`, `mx`, `txt` for common record types
- Supports A, AAAA, CNAME, MX, TXT, SRV, CAA, SSHFP, TLSA records

## Install

```bash
# Clone and symlink
git clone https://github.com/olibuijr/1984dns.git
ln -s "$(pwd)/1984dns/1984dns" ~/.local/bin/1984dns

# Or just copy the script
curl -o ~/.local/bin/1984dns https://raw.githubusercontent.com/olibuijr/1984dns/master/1984dns
chmod +x ~/.local/bin/1984dns
```

### Requirements

- `bash` 4+
- `curl`
- `grep` with `-P` (Perl regex) support — standard on Linux, install `grep` via Homebrew on macOS

## Configuration

Set your 1984.is credentials via environment variables or config file:

```bash
# Option 1: Environment variables
export DNS1984_USER="you@example.com"
export DNS1984_PASS="yourpassword"

# Option 2: Config file (recommended)
mkdir -p ~/.config/1984dns
cat > ~/.config/1984dns/config << EOF
DNS1984_USER=you@example.com
DNS1984_PASS=yourpassword
EOF
chmod 600 ~/.config/1984dns/config
```

## Usage

```bash
# List all zones on your account
1984dns zones

# List all records in a zone
1984dns list example.is

# Get a specific record
1984dns get example.is www A

# Create or update an A record
1984dns set example.is A www 1.2.3.4

# Quick A record update
1984dns point example.is www 1.2.3.4

# Set an MX record (priority 10)
1984dns mx example.is @ mail.example.is. 10

# Set a TXT record
1984dns txt example.is _dmarc '"v=DMARC1; p=reject"'

# Set a record with custom TTL and priority
1984dns set example.is MX @ mail.example.is. 3600 10

# Delete a record by entry ID (shown in list output)
1984dns del example.is 12345

# Refresh the zone cache
1984dns zones --refresh
```

## Commands

| Command | Description |
|---------|-------------|
| `zones [--refresh]` | List all zones on your account |
| `list <zone>` | List all records in a zone |
| `get <zone> <host> [type]` | Get specific record(s) |
| `set <zone> <type> <host> <value> [ttl] [pri]` | Create or update a record |
| `del <zone> <entry_id>` | Delete a record by entry ID |
| `point <zone> <host> <ip>` | Quick A record update |
| `mx <zone> <host> <value> [pri]` | Set MX record |
| `txt <zone> <host> <value>` | Set TXT record |

## How it works

1984.is doesn't expose a public API for DNS management. This tool reverse-engineers the web interface:

1. Authenticates via the same AJAX endpoint the web UI uses (`/api/auth/`)
2. Parses zone pages to extract record data from form fields
3. Submits record changes via the same POST endpoints (`/domains/entry/`, `/domains/delentry/`)
4. Caches session cookies and zone IDs locally for speed

Session cookies are stored in `~/.cache/1984dns/` and reused until they expire. Zone IDs are cached in `~/.cache/1984dns/zones.cache` — run `1984dns zones --refresh` to update after adding/removing zones in the web UI.

## Supported record types

A, AAAA, CNAME, MX, TXT, SRV, CAA, SSHFP, TLSA, NS

## Security

- Credentials are stored in `~/.config/1984dns/config` with `600` permissions
- Session cookies are stored in `~/.cache/1984dns/`
- No credentials are ever sent to any service other than 1984.is
- The tool uses HTTPS for all communication

## License

MIT

## About 1984.is

[1984 Hosting](https://1984.is) is an Icelandic green hosting company dedicated to privacy and freedom of speech since 2006. They offer shared hosting, VPS, FreeDNS, and domain registration — all powered by renewable Icelandic energy.

This tool is not affiliated with or endorsed by 1984 Hosting. It's a community tool built by a happy customer.
