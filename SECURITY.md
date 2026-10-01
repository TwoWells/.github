# Security Policy

This policy applies org-wide: it is the default for every TwoWells
repository that does not define its own security policy.

## Reporting a vulnerability

Use GitHub's private vulnerability reporting: on the affected repository,
**Security → Report a vulnerability**. Please do not disclose details in
public issues, discussions, or PRs before a fix ships.

## Release integrity

TwoWells releases follow the org-wide release contract documented in
[PROVENANCE.md](PROVENANCE.md): immutable release assets, a published
`.sha256` sidecar per asset, build-provenance attestations, and Trusted
Publishing for registries.

**A checksum mismatch is a security signal, not a nuisance.** If an install
fails hash verification (pacman, Homebrew, Scoop, or a manual
`sha256sum -c`), do not bypass the check and do not re-hash the download to
make it pass. See PROVENANCE.md's verification guidance, and report what you
found via the channel above.
