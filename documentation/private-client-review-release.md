# Mayadeen Event World — Private Client Review Release

## Scope

This profile packages the current five-platform Event World for a private,
authenticated client review. It is a static frontend release. It does not add
a backend, external integration, live IoT, simulation, or production authority.

The release preserves these truth boundaries:

- KAP operational readiness remains `cannot-determine`.
- V15 remains a received journey candidate, not an approved `SpatialRoute`.
- Rhino and Web3D content remains unregistered design information, not
  engineering or As-Built truth.
- No HSE, route, capacity, evacuation, opening, or live-operation approval is
  implied by packaging or hosting.

## Build And Verify

From a clean committed repository root, with the verified local runtime
derivatives staged, run the strict final gate before either package:

```bash
pnpm verify:final-event-world-release
EVENT_WORLD_REVIEW_ORIGIN=https://private-review.example/ pnpm package:final-event-world-convergence-review
pnpm package:private-client-review
pnpm verify:private-client-review
```

The strict gate writes `test-results/final-event-world-receipts.json` plus one
ignored, fingerprinted gate log per required command under
`.final-event-world-verification/gates/`. The separate root prevents Playwright
from deleting earlier gate evidence when it cleans `test-results/` for a new
browser run.
The final convergence package refuses a
missing, stale, partial, failed, dirty-worktree or wrong-HEAD receipt. It writes
its outer ZIP receipt to
`test-results/final-event-world-package-receipt.json`.
Each verification or packaging attempt clears its corresponding prior receipt
before validation, and each packaging attempt clears its target ZIP before
rebuilding it. ZIP existence without the matching successful current-HEAD
receipt is never release evidence.
`EVENT_WORLD_REVIEW_ORIGIN` should be the authenticated deployment base URL; the
packaged `reports/review-links.json` also carries each relative route and a
portable deployment template. If the variable is omitted, the direct links are
explicitly classified as local package-preview links.

The private package command performs a fresh production build, constructs a new
allowlisted frontend tree, verifies every file and expected runtime fingerprint,
writes `SHA256SUMS`, creates the ZIP twice, and requires both ZIP hashes to
match. Both final package commands first remove only the ignored generated
`dist/` and `dist-private-client-review/` directories so synchronized stale
copy artifacts cannot enter a release; tracked and private source assets are not
touched. For commit `<short-head>` it writes:

- `dist-private-client-review/` (verified deploy tree);
- `$HOME/Downloads/mayadeen-event-world-private-client-review-<short-head>.zip`;
- `test-results/private-client-review-package-receipt.json`;
- `test-results/private-client-review-verify-receipt.json`.

The allowlist contains only:

- `index.html` and Vite content-hashed assets;
- presentation assets used by the Arabic RTL application;
- all 33 manifest-bound D5 preview images;
- the 12 verified Design World GLB chunks and their verified runtime indexes;
- the two manifest-bound browser GLB derivatives and one preview;
- the four tracked, custody-safe runtime manifests.

PDF, Office, RAR, D5 project data, Rhino 3DM, CAD/BIM source formats, private
filesystem paths, secret-like values, contact PII, GPS metadata, symlinks, and
unlisted files are rejected.

## Static Host Contract

Deploy the contents of `frontend/` at the HTTPS origin root (`/`). The build
uses root-relative URLs and must not be mounted under a subdirectory without a
separate rebuild and verification for that base.

The host must:

1. require authenticated private access;
2. disable directory listing and deny dotfiles;
3. rewrite non-file SPA requests to `/index.html`;
4. serve `.glb` as `model/gltf-binary`;
5. serve Vite-hashed `/assets/*` immutably;
6. avoid long-lived caching for stable runtime-manifest and GLB paths;
7. apply the supplied security headers;
8. make no external-network exception for the frontend;
9. publish by atomic directory swap, never by mixing revisions.

The current owner-only review CSP temporarily permits `'unsafe-eval'` because
the existing Ajv validators compile schemas at runtime, and permits `blob:` in
`connect-src` for the verified local Web3D loader. A later hardening stage
should precompile Ajv validators and remove the eval exception before any
broader publication.

## Verification At The Host

After upload:

1. verify the ZIP SHA-256 before extraction;
2. verify every entry in the package-level `SHA256SUMS`;
3. confirm the root returns the committed `index.html` over HTTPS;
4. confirm a deep Event World URL receives the SPA fallback;
5. confirm JavaScript, CSS, PNG/WebP and GLB MIME types;
6. confirm CSP and the remaining security headers;
7. review all five platform routes with zero failed or external requests;
8. keep the prior complete package available as the rollback target.

Before upload, verify both locally generated ZIPs and their persisted receipts:

```bash
CURRENT_HEAD="$(git rev-parse HEAD)"
STRICT_RECEIPT="test-results/final-event-world-receipts.json"
jq -e --arg head "$CURRENT_HEAD" '
  .status == "passed" and .featureCommit == $head and
  .worktreeCleanBefore == true and .worktreeCleanAfter == true and
  .headUnchanged == true and .requiredGateCount == (.gates | length) and
  ([.gates[] | select(.status != "passed" or .exitCode != 0)] | length) == 0
' "$STRICT_RECEIPT"

WAVE_ZIP="$HOME/Downloads/mayadeen-event-world-final-convergence-private-review.zip"
WAVE_RECEIPT="test-results/final-event-world-package-receipt.json"
test "$(jq -r .featureCommit "$WAVE_RECEIPT")" = "$CURRENT_HEAD"
test "$(shasum -a 256 "$STRICT_RECEIPT" | awk '{print $1}')" = "$(jq -r .verificationReceiptSha256 "$WAVE_RECEIPT")"
test "$(shasum -a 256 "$WAVE_ZIP" | awk '{print $1}')" = "$(jq -r .sha256 "$WAVE_RECEIPT")"
unzip -t "$WAVE_ZIP"

PRIVATE_RECEIPT="test-results/private-client-review-package-receipt.json"
PRIVATE_ZIP="$(jq -r .zipPath "$PRIVATE_RECEIPT")"
test "$(jq -r .featureCommit "$PRIVATE_RECEIPT")" = "$CURRENT_HEAD"
test "$(shasum -a 256 "$PRIVATE_ZIP" | awk '{print $1}')" = "$(jq -r .sha256 "$PRIVATE_RECEIPT")"
unzip -t "$PRIVATE_ZIP"

VERIFY_ROOT="$(mktemp -d)"
unzip -q "$WAVE_ZIP" -d "$VERIFY_ROOT/wave"
WAVE_ROOT="$VERIFY_ROOT/wave/mayadeen-event-world-final-convergence-private-review"
test "$(shasum -a 256 "$WAVE_ROOT/package-manifest.json" | awk '{print $1}')" = "$(jq -r .packageManifestSha256 "$WAVE_RECEIPT")"
(cd "$WAVE_ROOT" && jq -r '.files[] | "\(.sha256)  \(.path)"' package-manifest.json | shasum -a 256 -c -)

unzip -q "$PRIVATE_ZIP" -d "$VERIFY_ROOT/private"
PRIVATE_SHORT_COMMIT="$(jq -r .featureCommit "$PRIVATE_RECEIPT" | cut -c1-12)"
PRIVATE_ROOT="$VERIFY_ROOT/private/mayadeen-event-world-private-client-review-$PRIVATE_SHORT_COMMIT"
test "$(shasum -a 256 "$PRIVATE_ROOT/SHA256SUMS" | awk '{print $1}')" = "$(jq -r .treeSha256 "$PRIVATE_RECEIPT")"
(cd "$PRIVATE_ROOT" && shasum -a 256 -c SHA256SUMS)
PRIVATE_CLIENT_REVIEW_VERIFY_RECEIPT=test-results/private-client-review-extracted-verify-receipt.json \
  pnpm exec tsx scripts/verify-private-client-review.ts --root="$PRIVATE_ROOT/frontend"
```

The Wave package manifest intentionally excludes only itself; therefore its
`filesCovered` value is one less than the extracted regular-file count. The
private package `SHA256SUMS` intentionally excludes only itself. Preserve the
two package receipts, the strict gate receipt/logs, both ZIP hashes, and the
post-extraction verification receipt together as the release evidence set.

## Rollback

Rollback replaces the complete hosted directory with the previously verified
release. Do not copy individual files between releases because stable GLB and
runtime-manifest paths are intentionally revalidated per release.
