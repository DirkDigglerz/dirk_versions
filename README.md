# dirk_versions

Public version manifest for DirkScripts resources (including private ones).

`versions.json` is the source of truth: a flat `{ resource_name: "X.Y.Z" }` map.
Consumed at runtime by `lib.versionCheck` (in `dirk_lib`) when called with
`{ manifest = true }`:

```lua
-- server-side, in any DirkScripts resource
lib.versionCheck('DirkDigglerz/dirk_versions', { manifest = true })
```

`lib.versionCheck` fetches `https://raw.githubusercontent.com/DirkDigglerz/dirk_versions/main/versions.json`,
looks up the invoking resource's name as the key, and prints an orange console
warning if the locally installed `fxmanifest.lua` `version` is older.

Other devs can run their own manifest repo and point `lib.versionCheck` at it:

```lua
lib.versionCheck('SomeOtherDev/their_versions', { manifest = true })
```

## How a resource registers/updates its version

The consumer repo's release workflow fires a `repository_dispatch` event at
this repo with `event_type: bump-version` and a payload of
`{ resource, version }`. The receiver workflow (`bump-version.yml`) validates
the payload, updates `versions.json`, and commits.

The dispatch needs a PAT with `repo` scope on this repo, stored as a secret
in the consumer repo (e.g. `DIRK_VERSIONS_PAT`).

## Why a separate repo

- One public source of truth lets `lib.versionCheck` work for **private**
  resources, since `raw.githubusercontent.com` doesn't require auth for public
  files.
- Keeps version-tracking changes out of the consumer repo's commit history.
- Other devs can fork this layout for their own manifest if they prefer not
  to share one.
