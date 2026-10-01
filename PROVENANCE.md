# Release Provenance & Contract

The canonical statement of what every TwoWells release publishes, what is
guaranteed about those bytes, and how anyone — a user or a package
maintainer — verifies them.

This is the producer-side counterpart of the consumer-side policies our
distribution repos publish: how *they* decide which upstream bytes to trust
([pkgbuilds/PROVENANCE.md](https://github.com/TwoWells/pkgbuilds/blob/main/PROVENANCE.md),
[scoop-bucket/PROVENANCE.md](https://github.com/TwoWells/scoop-bucket/blob/main/PROVENANCE.md),
[homebrew-tap/CONTRIBUTING.md](https://github.com/TwoWells/homebrew-tap/blob/main/CONTRIBUTING.md)).
When those documents cite "the TwoWells release contract," this is the
document they mean. Each project's release workflow may restate the contract
in its header for self-containedness; this document is authoritative when
they disagree.

## Guarantees

1. **Release assets are immutable.** Once uploaded to a GitHub Release, an
   asset's bytes never change. An asset that hashes differently later is a
   broken release or tampering — never routine drift.
2. **Every asset ships with a published hash claim.** A `sha256sum`-format
   `<asset>.sha256` sidecar is uploaded next to each asset. Consumers copy
   the claim; nothing downstream ever "computes" a pin by hashing its own
   download.
3. **Archives carry build provenance.** Release archives are attested with
   [GitHub build provenance](https://docs.github.com/en/actions/security-for-github-actions/using-artifact-attestations)
   (`actions/attest-build-provenance`), linking the bytes to the exact
   workflow run, commit, and builder that produced them. Sidecars are not
   attested — they are claims about the archives, not artifacts themselves.
4. **Registry publishing uses Trusted Publishing.** Crates are published to
   crates.io via OIDC (no long-lived stored tokens). Release workflows
   trigger only on tag pushes, which require write access, so fork PRs can
   never reach publish credentials.

## The release contract

Per tagged release, each project publishes:

1. **Tag `vX.Y.Z`** — CI enforces that the tag equals the manifest version.
2. **One prebuilt archive per platform**: `<name>-<rust-triple>.tar.gz`
   (`.zip` on Windows), with the binary at the archive root.
3. **A source tarball**: `<name>-src-X.Y.Z.tar.gz` — a `git archive` of the
   tagged tree with a codeload-compatible `<Repo>-X.Y.Z/` prefix. This is
   the artifact source builds (e.g. the AUR `<name>` package) pin against;
   GitHub's on-the-fly codeload archives are not byte-stable and carry no
   published hash.
4. **A `.sha256` sidecar for every asset above.**
5. **In the tagged tree**: the committed lockfile (source builds run
   `--locked`), the `LICENSE` file, and a working
   `<name> completions {bash,zsh,fish}` subcommand (packages build shell
   completions by running the binary).

Breaking any of these silently breaks the AUR/Homebrew/Scoop bump automation
on the next release, which is why asset naming stays stable even when it is
inconvenient.

## Verifying a release

Checksum — compare the asset against its published sidecar:

```sh
sha256sum -c themis-x86_64-unknown-linux-gnu.tar.gz.sha256
```

Provenance — confirm the asset was built by the project's release workflow
on GitHub's infrastructure:

```sh
gh attestation verify themis-x86_64-unknown-linux-gnu.tar.gz --repo TwoWells/Themis
```

If a checksum does not match, do not re-download until it does and do not
"fix" a pin by re-hashing: establish which side changed. Fetch the asset via
a second network path, compare both against the sidecar, and report what you
find (see [SECURITY.md](SECURITY.md)).

## Where the contract is enforced

- Each project's release workflow produces the assets and restates the
  contract in its header comment.
- Upstream preflights (e.g. `make release-contract`) check the
  locally-verifiable parts before a tag is pushed.
- Downstream, the bump automation hard-fails on a missing sidecar rather
  than hashing a download, and checksum verification runs on every build —
  see the distribution repos' policies linked above.
