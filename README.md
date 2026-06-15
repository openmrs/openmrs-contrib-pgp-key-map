# OpenMRS PGP Keys Map

Trust policy for verifying the PGP signatures of OpenMRS Maven artifacts and the
third-party dependencies the OpenMRS platform pulls in.

This repo publishes a single artifact, `org.openmrs.maven:openmrs-pgp-key-map`,
a jar containing one file: [`pgp-key-map.list`](pgp-key-map.list). The OpenMRS
module parent POM
([`maven-parent-openmrs-module`](https://github.com/openmrs/openmrs-contrib-maven-parent-module))
adds that jar as a dependency of
[`pgpverify-maven-plugin`](https://www.simplify4u.org/pgpverify-maven-plugin/)
and loads the file from the plugin classpath, so every module build verifies its
dependencies against this map.

## What the map says

Format reference:
<https://www.simplify4u.org/pgpverify-maven-plugin/keysmap-format.html>

- **`org.openmrs.*`** must be signed by the OpenMRS Bot code-signing key,
  fingerprint `CA12619FDE8CD6A93FAFE458A6F9608DCC73473F`
  ([keyserver](https://keyserver.ubuntu.com/pks/lookup?search=CA12619FDE8CD6A93FAFE458A6F9608DCC73473F&fingerprint=on&op=index)).
- Third-party artifacts that the community map
  (`org.simplify4u:pgp-keys-map`) does not already cover are pinned here, each
  scoped to the exact version the platform depends on.
- A handful of old artifacts are unsigned (`noSig`) or carry broken signatures
  (`badSig`); those are listed explicitly so the rest of the map can stay
  strict.

Entries marked with a `TODO` are temporary `noSig` allowances for OpenMRS
artifacts that the signature backfill missed — remove them once those artifacts
are signed.

## Updating the map

1. Edit `pgp-key-map.list`. To discover the key for a new artifact:
   ```
   mvn org.simplify4u.plugins:pgpverify-maven-plugin:show -Dartifact=groupId:artifactId:version
   ```
   Leave `<version>` in `pom.xml` at the current `-SNAPSHOT` — the release
   workflow sets the release version, you don't bump it by hand.
2. Open a PR. CI builds the jar; the keys map is exercised end-to-end by the
   parent module's integration tests.
3. After merge, run the **Release** workflow (below) to publish a new version.
4. Bump the `openmrs-pgp-key-map` dependency version in the parent module POM to
   pick up the new release.

> Note: pgpverify caches validation results in
> `target/pgpverify-prior-validation-checksum`, keyed on the *set of artifacts*,
> not the keys map contents. After editing the map locally, run `mvn clean` (or
> delete that file in the consuming build) or the change is silently ignored.

## Release

The POM is kept at `X.Y.Z-SNAPSHOT`. The **Release** workflow
(`workflow_dispatch`) runs `maven-release-plugin` via the shared OpenMRS release
script: it strips `-SNAPSHOT`, tags and deploys the release version to
`mavenrepo.openmrs.org`, then commits the next development version. Inputs:
`releaseVersion` (e.g. `1.0.0`), optional `developmentVersion`, and `skipTests`.

Required secrets:

| Secret | Purpose |
| --- | --- |
| `OMRS_RELEASE_BOT_APP_ID` / `OMRS_RELEASE_BOT_PRIVATE_KEY` | GitHub App used to push the release commits and tag. |
| `MAVEN_REPO_USERNAME` / `MAVEN_REPO_API_KEY` | Credentials for `mavenrepo.openmrs.org`. |
| `MAVEN_GPG_PRIVATE_KEY` / `MAVEN_GPG_PASSPHRASE` | Optional — set to GPG-sign the released jar (activates the signing profile). |