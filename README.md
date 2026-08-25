# alice-digitalocean

Desired state for one Alice Transmission server on DigitalOcean. Transmission's
web UI binds only to the Droplet's loopback interface and is reachable through
the managed SSH alias.

```sh
./green validate
./green build
./green create --dry-run
./green create
./green sync
./green describe
./green tunnel 19091
```

While the foreground tunnel runs, open
`http://127.0.0.1:19091/transmission/web/`. A successful create performs the same
tunneled HTTP check before returning. `sync` prints this URL and keeps its tunnel
open while every desired torrent downloads. Completed data is copied
directly into `~/Downloads/alice`; after a final checksummed rsync succeeds, the
Droplet is destroyed. `transmission-magnet-links` is empty here, so `sync`
has nothing to wait for and tears the Droplet down after one copy — use
`create` and `tunnel` when the intent is a UI to keep open.

Credentials belong only in `.envrc.private` as `COLORS_PAR_DO_TOKEN`. Never set
`COLORS_PAR_PROFILE`. Keep `compute-prevent-destroy: true`.
`COLORS_PAR_COMPUTE_PREVENT_DESTROY` is ignored: `delete` itself authorizes a
manual destroy, and `sync` authorizes destruction only after its final copy
succeeds.

The deployment uses local OpenTofu state. Retain `.colors/` until deletion has
completed, but never edit, read as source, or commit it.
