# `automation` — the branch that keeps `main` honest

This branch exists to hold one workflow. It is the fork's default branch, and that is deliberate,
not an accident of setup.

`main` on this fork is a mirror of [`swgoh-utils/gamedata`](https://github.com/swgoh-utils/gamedata)
and needs to stay byte-identical to it, so that syncing is always a fast-forward and a local
`git pull` never conflicts. Committing anything at all to `main` — including the workflow that
does the syncing — diverges it from upstream permanently, and from then on nothing ever
fast-forwards again.

GitHub only fires `schedule` on a repository's default branch. So the workflow has to live on
the default branch, and the mirror must not be it. Hence this split:

| Branch | Holds | Diverges from upstream? |
| --- | --- | --- |
| `automation` (default) | [`.github/workflows/sync-upstream.yml`](.github/workflows/sync-upstream.yml) | Unrelated to it entirely |
| `main` | the gamedata feed, mirrored | Never |

## Cloning

`git clone` lands you on `automation`, which is not what you want for the data:

```sh
git clone -b main https://github.com/MarTrepodi/swgoh_gamedata.git
```

## What the workflow does

Every six hours it compares `main` against upstream. If there is nothing new it stays silent. If
there is, it fast-forwards `main` and posts the new versions to Discord along with the `git pull`
you now owe your local clone — the whole point, since a mirror that advances on GitHub leaves a
local clone stale with no visible symptom until something reads the wrong pull.

If `main` ever has diverged, it opens a pull request instead of merging unattended.

## Configuration

| Kind | Name | Value |
| --- | --- | --- |
| Secret | `DISCORD_WEBHOOK_URL` | Discord: Server Settings > Integrations > Webhooks > Copy Webhook URL |
| Variable | `SYNC_BRANCH` | `main` |

Upstream is read from GitHub's own fork relationship, so it is not configured here.
