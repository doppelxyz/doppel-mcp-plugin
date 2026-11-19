# Releasing

Maintainer notes. Releases are manual by design: this repository has no build step
and GitHub Actions is disabled, so a release is a signed tag plus a GitHub Release on
`main`.

## Rules in force

- `main` accepts only pull requests with one code-owner approval, signed commits,
  linear history, and squash merges (ruleset "Protect main").
- Tags matching `v*` cannot be deleted, moved, or created unsigned (ruleset
  "Protect release tags"). A published version is permanent; fix forward with a new
  version.
- The `version` field in `plugin.json` and `.claude-plugin/plugin.json` must equal the
  tag being cut.
- Every pull request carries the release checklist from
  `.github/PULL_REQUEST_TEMPLATE.md`; whoever merges cuts the tag.

## Cutting a version

1. Land the change on `main` through a pull request. If anything user-visible changed,
   bump `version` in both manifests in that same pull request (patch for docs or
   wording, minor for a change to what the plugin bundles or to manifest
   capabilities).
2. On an up-to-date `main`, create the signed tag at the approved commit and push it:

   ```sh
   git switch main && git pull --ff-only
   git tag -s vX.Y.Z -m "Doppel plugin vX.Y.Z"
   git push origin vX.Y.Z
   ```

3. Publish the GitHub Release from the tag, with a short note of what changed:

   ```sh
   gh release create vX.Y.Z --title "vX.Y.Z" --notes "..."
   ```

4. If the change affects what a directory reviewer would see (manifests, README),
   resubmit to the affected directories (Anthropic, Cursor) pointing at the new tag.
   ChatGPT does not consume this repository.

## Signing

Signing uses SSH keys (`gpg.format = ssh`). Register your public key on GitHub under
*Settings → SSH and GPG keys → Signing keys*, then in this clone:

```sh
git config gpg.format ssh
git config user.signingkey ~/.ssh/<key>.pub
git config commit.gpgsign true
git config tag.gpgsign true
```

GitHub shows the commit or tag as *Verified* only if the public key is registered as a
signing key on the account that authored it.
