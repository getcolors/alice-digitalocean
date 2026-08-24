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
./green sync                   # authorized ephemeral download lifecycle
./green describe
./green tunnel 19091
./green delete                 # guarded, destructive, authorization required
```

Build and dry-run work without credentials. Never run real create/delete without
explicit authorization. Never read `.envrc.private` or edit/read `.colors/` as
source.

## Desired state and credentials

`colors.yml` is the only normal edit and contains non-secret values. The
DigitalOcean token is `COLORS_PAR_DO_TOKEN` in ignored `.envrc.private`. A
failing credential check names what actually failed: only 401 and 403 mean the
token, while a 5xx is DigitalOcean's own outage and explicitly says not to
rotate anything. Never
export `COLORS_PAR_PROFILE`; it can redirect OpenTofu state. Preserve
`compute-prevent-destroy: true`. `COLORS_PAR_COMPUTE_PREVENT_DESTROY` is ignored;
explicit `delete` authorizes manual destruction, while `sync` authorizes it only
after all desired torrents complete and the final rsync succeeds.

The deployment carries neither a VPC UUID nor an SSH key id, because both are
optional and presence is the only switch. The region's default VPC is
discovered at runtime through a `digitalocean_vpc` data source, and the apply
asserts it really is the account default. The machine keypair is generated and
owned by the package as `~/.ssh/alice-digitalocean`(`.pub`), registered with
DigitalOcean under the profile name, and removed only after the compute destroy
succeeds. Restore either key to desired state to pin it by hand instead. One
Ubuntu Droplet and local OpenTofu state. `sync` adds the configured Kali magnets, keeps the UI
tunnel open, copies download-directory contents directly into
`~/Downloads/alice`, and destroys the Droplet only after a final checksummed
copy. Any failure retains the deployment for a retry. Retain `.colors/` until
deletion is complete. The UI
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

Because the package owns the keypair, an existing `~/.ssh/alice-digitalocean`
with no readable compute state stops the run rather than being overwritten — it
may be the only credential to a Droplet that is still alive. A `sync`
interrupted between the compute destroy and the key removal leaves exactly that
state. Verify at DigitalOcean that no Droplet for the profile survives, then
remove `~/.ssh/alice-digitalocean` and `~/.ssh/alice-digitalocean.pub` and
retry. Never adopt a keypair by deleting that check.

## Documentation

`index.html` is this repository's landing page and carries two analytics tags:
GA4 measurement ID `G-4VKP1WY4QJ`, whose explicit `page_title` must exactly
equal the decoded HTML `<title>` and stay distinct and stable so one Analytics
property can separate repositories, and the self-hosted Rybbit snippet
`<script src="https://rybbit.getcolors.ai/api/script.js" data-site-id="9fb9c41a6d49" defer></script>`,
which shares one site ID across every page because `getcolors.github.io/<repo>/`
paths already encode the repository. Never add one tag without the other.

## Git

Work on the current branch. Do not commit or push unless explicitly authorized.
