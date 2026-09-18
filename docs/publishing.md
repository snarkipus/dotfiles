# Publishing a snapshot

This public repository is intentionally separate from the private authoring
repository. Publish only explicitly reviewed exports; never make the authoring
repository public or copy its history here.

## Initial upload

A draft `v0.1.0` release reserves a place for the initial configuration bundle.
The repository owner can open GitHub > Releases > Edit the draft and attach:

- `company-shell.tar.gz`
- `company-shell.tar.gz.sha256`
- `company-shell.manifest.txt`

These names are the current export convention, not a requirement to use the
configuration on a company machine. Do not nest the files in another archive,
rename their format to evade filters, or publish encoded archive content as text.

Before publication:

1. Inspect the exact archive contents, not only the working export folder.
2. Include only the intended `home/` paths: adapted configs and selected plugins.
3. Exclude credentials, keys, personal identity, workstation host paths, Git
   metadata, caches, histories, saved sessions, and the export's `originals/`.
4. Preserve third-party license notices. Record plugin upstream URLs and exact
   revisions only when verified; do not label copied live trees as pinned merely
   because the authoring configuration declares a pin.
5. Check permissions, file types, contained links, and the explicitly documented
   tmux-resurrect test-link exception.
6. Verify the SHA-256 file and test extraction in a fresh temporary directory.
7. Record which exact package versions and tests passed and which remain pending.
8. Update the README's availability/status text when assets become public.

A secret-pattern scan is useful, but is not a substitute for reviewing the
allowlisted contents. Do not include host-local absolute paths or corporate
identifiers in public logs or provenance.

### CLI upload from the machine holding the export

From the directory containing the reviewed files, with GitHub CLI authenticated
as a repository maintainer:

```sh
sha256sum -c company-shell.tar.gz.sha256
gh release upload v0.1.0 \
  company-shell.tar.gz \
  company-shell.tar.gz.sha256 \
  company-shell.manifest.txt \
  --repo snarkipus/dotfiles
```

This attaches assets to the existing draft; it does not publish the release.
No `--clobber` is used. Resolve conflicting assets deliberately rather than
silently replacing a published download.

Inspect the uploaded draft in GitHub. Update release notes with the actual
validation results, then publish using the web UI. If compatibility remains
partial, mark it as a prerelease and say what remains untested. The release tag
identifies this repository's instructions; record the export's own source and
plugin provenance separately, without exposing private material.

After publishing, download the assets again and verify the archive checksum.
A successful upload request alone is not verification of the downloadable result.

## Future releases

Use a new version and explicit asset names, retain the prior working release,
and document configuration changes and supported runtime versions. Avoid bundling
package-manager installers or automatic network fetches into shell startup.
Original source maintenance can stay in Chezmoi; recipients consume snapshots
without needing Chezmoi or write access to the source repository.
