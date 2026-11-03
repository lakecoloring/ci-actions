# ci-actions

Composite GitHub Actions shared by the lakecoloring CI workflows.

## wipe-and-retry

Wipes the runner workspace and re-runs `actions/checkout` after a checkout
fails on a poisoned `.git`. See `wipe-and-retry/action.yml` for when and how
to use it.

```yaml
- name: Wipe and retry on checkout failure
  if: steps.checkout.outcome == 'failure'
  uses: lakecoloring/ci-actions/wipe-and-retry@<sha> # v1
```

## Why a separate repository

The runner downloads every action a job references when the job starts, even
for steps that end up skipped. This action used to live in `lake-app`, so
every job downloaded the whole `lake-app` tarball (about 63 MB) from
codeload just to load one small file. All CI Macs share one public address,
which made codeload rate-limit them (`429 Too Many Requests`).

Here the tarball is a few KB. Pinned by SHA, it is also served from the
runners' local action archive cache (`lake-app/Setup/action_cache.yml`), so
most jobs do not download it at all.

## Why it is public

The action archive cache is filled without a token, and keeping the repo
public avoids per-repo access settings. Keep secrets and anything
Lake-specific out of it.

## Releasing

Pin callers to a commit SHA with the tag as a comment, like every other
action in `lake-app`. After a change, cut a new tag (`v1.1`, or `v2` for a
breaking change) and bump the pins; Renovate picks up new tags.
