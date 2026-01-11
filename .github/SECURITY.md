# Security Policy

## Reporting a Vulnerability

If you discover a security vulnerability, please email `security at mail dot as93 dot net` directly. Do not open a public issue.
You can expect a response within 48 hours. We'll work with you to understand and fix the issue promptly.

## Supported Versions

| Version | Supported          |
| ------- | ------------------ |
| v1.x    | ✅ Yes             |
| < v1.0  | ❌ No               |

## Security Best Practices

**When using this action:**
- Never commit SSH private keys to your repository
- Always use GitHub Secrets for sensitive data
- Use deploy keys with minimum required permissions (write access only to target repo)
- Rotate SSH keys periodically
- Review the action's code before using in production

**SSH Key Management:**
- Generate unique SSH keys for each mirror target
- Use ED25519 keys when possible (`ssh-keygen -t ed25519`)
- Store private keys only in GitHub Secrets
- Revoke old keys when no longer needed

## What We Do
- SSH private keys are passed via environment variables (safe for multi-line secrets)
- Keys are validated before use
- Stored in unique temp files with 0600 permissions
- Automatically cleaned up after execution
- GitHub Actions automatically masks secrets in logs
- Host keys retrieved via `ssh-keyscan` and verified
- `StrictHostKeyChecking` enforced to prevent MITM attacks
- Action fails if host keys cannot be verified
