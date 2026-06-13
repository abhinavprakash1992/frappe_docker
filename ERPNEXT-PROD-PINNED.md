# erpnext-prod — commit-pinned ERPNext v15 image

Reproduces the `erpnext-prod` bench (migrated to Docker on a free Canonical VM) **exactly**:
every app is pinned to the precise commit running in production, and prod's uncommitted
working-tree customizations are re-applied as patches.

## Pinned apps (see `apps-pinned.txt`; frappe base via build arg)
frappe 665fce2 · erpnext 1d14ba1 · india_compliance d887c82 · payments 3cebd94 · crm b131d36 ·
insights 3448944 · telephony d4ee5b4 · ecommerce_integrations ba8ac50 · frappe_whatsapp 0a84e43 · helpdesk db5b0cf

## Customizations carried (`patches/`)
- `erpnext_serial_no.patch` — Serial No DocType delivery fields
- `crm_auto_imports.patch` — crm frontend auto-imports

## Build (context = repo root)
```
DOCKER_BUILDKIT=1 docker build -f images/custom/Containerfile.pinned -t erpnext-pinned:v15 .
```
Runtime baked in: Python 3.11.9, Node 20.20.2, wkhtmltopdf 0.12.6, MariaDB client. Pinning uses
`git fetch <url> <sha>` (a raw commit SHA can't be used with `git clone --branch`).

## Run
```
echo "DB_ROOT_PASSWORD=$(openssl rand -hex 24)" > .env
docker compose --env-file .env -f erpnext-prod-compose.yaml up -d           # staging (no scheduler/workers)
docker compose --env-file .env -f erpnext-prod-compose.yaml --profile cutover up -d   # go-live (adds scheduler+workers)
```
NOTE: MariaDB `max_allowed_packet=1G` is required (prod data has a >16MB row); set in the compose `db` command.
Restore prod data with `bench --site <site> restore <db.sql.gz> --with-public-files ... --with-private-files ...`
and copy prod's `encryption_key` into the site config.
