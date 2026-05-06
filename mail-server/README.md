# Mail Server — mattjack.cloud

Self-hosted email on the mattjack.cloud VPS, providing `jacko@mattjack.cloud`.

## Server

| | |
|---|---|
| Host | srv1530832.hstgr.cloud (Hostinger KVM 2) |
| IPv4 | 76.13.20.82 |
| OS | Ubuntu 24.04 LTS |
| Mail hostname | mail.mattjack.cloud |

## Stack

| Component | Package | Purpose |
|---|---|---|
| MTA | Postfix 3.8.6 | Send/receive SMTP |
| IMAP | Dovecot 2.3.21 | IMAP access + SASL auth for Postfix |
| DKIM | OpenDKIM 2.11.0 | Sign outbound mail |
| TLS | Let's Encrypt (certbot) | TLS cert for mail.mattjack.cloud |

## Mailboxes

| Address | Linux user | Maildir |
|---|---|---|
| jacko@mattjack.cloud | jacko | /home/jacko/Maildir |
| matt@mattjack.cloud | matt | /home/matt/Maildir |

## DNS Records

| Type | Name | Value |
|---|---|---|
| A | mail | 76.13.20.82 |
| MX | @ | mail.mattjack.cloud (priority 10) |
| TXT | @ | v=spf1 ip4:76.13.20.82 mx ~all |
| TXT | mail._domainkey | v=DKIM1; h=sha256; k=rsa; p=... |
| TXT | _dmarc | v=DMARC1; p=quarantine; rua=mailto:jacko@mattjack.cloud |
| PTR | 76.13.20.82 | mail.mattjack.cloud |

## Ports (UFW open)

| Port | Protocol | Purpose |
|---|---|---|
| 25 | TCP | SMTP inbound |
| 587 | TCP | Submission (authenticated send) |
| 465 | TCP | SMTPS |
| 993 | TCP | IMAPS |

## Client Settings

| | |
|---|---|
| IMAP server | mail.mattjack.cloud |
| IMAP port | 993 (SSL) |
| SMTP server | mail.mattjack.cloud |
| SMTP port | 587 (STARTTLS) |
| Username | jacko |

## Key File Locations

```
/etc/postfix/main.cf          # Postfix main config
/etc/postfix/master.cf        # Postfix service config
/etc/postfix/virtual          # Virtual alias map
/etc/dovecot/dovecot.conf     # Dovecot config
/etc/opendkim.conf            # OpenDKIM config
/etc/opendkim/keys/mattjack.cloud/mail.private  # DKIM private key (not in repo)
/etc/letsencrypt/live/mail.mattjack.cloud/      # TLS cert (auto-renews)
```

## Notes

- DKIM private key is NOT committed to the repo — back it up separately
- OpenDKIM uses a TCP socket (`inet:localhost:8891`) rather than a Unix socket to avoid Postfix chroot issues
- PTR record set via Hostinger support chat (not available in hPanel UI)
- Cert auto-renews via certbot systemd timer

## Setup Steps (for reference)

1. `apt install postfix dovecot-core dovecot-imapd dovecot-lmtpd opendkim opendkim-tools mailutils`
2. Generate DKIM keys: `opendkim-genkey -b 2048 -d mattjack.cloud -D /etc/opendkim/keys/mattjack.cloud -s mail`
3. Write configs (see `configs/` directory)
4. Issue cert: `certbot certonly --nginx -d mail.mattjack.cloud`
5. Add DNS records (A, MX, SPF, DKIM, DMARC)
6. Request PTR via Hostinger support
7. Open UFW ports: 25, 587, 465, 993
8. `systemctl restart opendkim postfix dovecot`
