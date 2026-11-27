# sublayer-maps

The Sublayer controller map library: one TOML file per controller, checked on every pull
request by the same rules the app uses, and published to `releases.sublayer.app/controllers/` on
merge (D-226). The app looks a controller up here only when the user turns on
LOOK UP NEW CONTROLLERS ONLINE or presses CHECK ONLINE (off by default, D-150).

> This folder is the **template** for the public repository `github.com/sublayer-app/sublayer-maps`
> (D-244: a dedicated organisation, created by the owner on github.com). It lives in the app
> repository (`tools/sublayer-maps/`) until then. `scripts/prepare-sublayer-maps-repo.sh`
> assembles the repository (these files and the shipped maps) for a one-step
> `gh repo create … --push`.

## Adding or changing a map

1. Put the map in `maps/`, named `<vendor>_<model>.toml`: lowercase, words joined with `_`.
   The library id is the file name with `-` for `_`, and it never changes once published.
2. Follow profile format 1 (`_docs/controllers/` in the app repository). A library map must have:
   - `status = "community"`. Only maintainers set `"verified"`, after testing on the hardware.
   - `version`: 1 for a new map. **Any change to the file's bytes needs a higher version**
     (D-233), so a copy cached at the old version is replaced.
   - `ports` or an `inquiry` match (port patterns at least 3 characters).
   - No `output_approved`. Only the user's ALLOW approves a map's start-up messages.
3. Open a pull request. The **check** job runs `sublayer-map-check` on the changed maps:
   - It fails on anything the app would refuse.
   - It also fails on keys the current build doesn't know, which the app would drop.

### Maps for controllers Sublayer ships a map for

A map named like a built-in map (for example "Numark Mixtrack Go") is only taken as an
**update** of the built-in one (D-237). It must:
- have a version strictly newer than the built-in (at most 1000 past it);
- keep the built-in map's ports, excluded ports and inquiry exactly.

The app keeps the built-in map's approval only if the update's start-up, shutdown and LED
messages are byte-identical; otherwise the user is asked again.

To **withdraw** a bad update, set the map back to the built-in version. Deleting it withdraws
nothing: the app keeps a copy it already cached.

## The checker

The Action downloads `sublayer-map-check`, a universal macOS binary built from the app
repository by `scripts/build-map-check.sh` and published at
`releases.sublayer.app/tools/`. Its file name and SHA-256 are pinned in `maps.yml`, so a fork's
pull request is checked without secrets and without the app's source. When the app's map rules
change, a new checker is built and published, and both pins move forward together.

## Checking locally

With the checker (`sublayer-map-check check --deny-warnings --maps-dir maps <files>`; a changed
map as `--against <its earlier version> <file>`), or from the app repository:

```sh
cargo run -p sublayer-app --bin sublayer-map-check -- check --deny-warnings --maps-dir <this repo>/maps <changed files>
cargo run -p sublayer-app --bin sublayer-map-check -- index <this repo>/maps <out folder>
```

`index` writes `controllers/index.json` and `controllers/<id>-v<n>.toml`: exactly what is
published.

## Publishing

On merge to `main`, the **publish** job rebuilds the index and uploads it as a build artifact.
The upload to `releases.sublayer.app` happens through Miranium, as releases do. The workflow
keeps that step disabled until its credentials are set up as repository secrets. After an
upload, check each served file's SHA-256 against `index.json`.
