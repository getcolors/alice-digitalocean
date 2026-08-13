# alice-digitalocean

Desired state for one Alice Transmission server on DigitalOcean. Transmission's
web UI binds only to the Droplet's loopback interface and is reachable through
the managed SSH alias.

```sh
./green validate
./green build
./green create --dry-run
./green create
./green describe
./green tunnel 19091
```

While the foreground tunnel runs, open
`http://127.0.0.1:19091/transmission/web/`. A successful create performs the same
tunneled HTTP check before returning.

Credentials belong only in `.envrc.private` as `COLORS_PAR_DO_TOKEN`. Never set
`COLORS_PAR_PROFILE`. Keep `compute-prevent-destroy: true`. For a separately
authorized deletion only:

```sh
COLORS_PAR_COMPUTE_PREVENT_DESTROY=false ./green delete
```

The deployment uses local OpenTofu state. Retain `.colors/` until deletion has
completed, but never edit, read as source, or commit it.
