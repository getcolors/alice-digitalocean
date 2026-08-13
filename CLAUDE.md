# CLAUDE.md

## What this repository is

Desired state for `alice-digitalocean`: one DigitalOcean Droplet in Amsterdam,
configured by the installed Alice Package Skill as a Transmission server. The
web UI is private to loopback and is verified through an SSH local forward.
Behavior lives in `../alice`; this repository contains configuration and an
installed launcher copy.

## Commands

```sh
./green validate
./green build
./green create --dry-run
./green create                 # authorization required
./green describe
./green tunnel 19091
./green delete                 # guarded, destructive, authorization required
```

Build and dry-run work without credentials. Never run real create/delete without
explicit authorization. Never read `.envrc.private` or edit/read `.colors/` as
source.

## Desired state and credentials

`colors.yml` is the only normal edit and contains non-secret values. The
DigitalOcean token is `COLORS_PAR_DO_TOKEN` in ignored `.envrc.private`. Never
export `COLORS_PAR_PROFILE`; it can redirect OpenTofu state. Preserve
`compute-prevent-destroy: true` and lift it only for one authorized delete with
`COLORS_PAR_COMPUTE_PREVENT_DESTROY=false`.

The deployment uses an existing Amsterdam VPC and SSH key, one Ubuntu Droplet,
and local OpenTofu state. Retain `.colors/` until deletion is complete. The UI
is loopback-only and uses SSH as its authentication boundary. The package also
disables Ubuntu 24.04's broken Transmission AppArmor notify profile. Use
`./green tunnel 19091`, then open
`http://127.0.0.1:19091/transmission/web/`.

## Installed launcher

The root `green` and `.agents/skills/package-alice-green/green` are copies, not
symlinks. Keep them byte-identical after every skill update. Never hand-edit the
stamped package SHA. For working-tree development use
`ALICE_LIB_ROOT=../alice ./green build`.

## Verification and recovery

Create must finish its tunneled UI acceptance check. `./green describe` reports
the alias, SSH, and service status. If Transmission is unhealthy, inspect
`systemctl status transmission-daemon` and its journal over SSH. If state is
lost, import or recover the Droplet state before deletion; do not guess a cloud
resource address.

## Git

Work on the current branch. Do not commit or push unless explicitly authorized.
