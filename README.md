# Claude sandbox + Bifrost

Tested with sbx **v0.50.1**, Bifrost **v2.3.2**, and Claude workload **2.1.267**.
Run commands on the Mac host from the repository root.

## Setup

First launch only: generate the dashboard setup token. Skip if `.env` exists.

```bash
openssl rand -hex 32 | sed 's/^/BIFROST_SETUP_TOKEN=/' > .env
chmod 600 .env
```

```bash
docker compose up -d
curl --fail http://localhost:8080/health
```

At http://localhost:8080, use the token from `.env` to create an admin account.
Add your Anthropic provider key and create a Bifrost virtual key with access to
`claude-sonnet-4-5`. Reuse existing configuration if already set up.

Store the **Bifrost virtual key** when prompted:

```bash
sbx secret set bifrost --sandbox claude-bifrost-localhost
```

## Run

```bash
sbx env run
```

Approve the plan. Choose **Yes** if Claude asks to use `local-placeholder`.
Use the same command for later sessions. The sandbox is `claude-bifrost-localhost`;
only `workspace/` is mounted. Change `workspace` in `sbxenv.yaml` before creation
to use another project.

## Credential mapping

```text
Claude → host.docker.internal:8080/anthropic → sbx proxy → localhost:8080 → Anthropic
```

- `sbxenv.yaml` selects Claude, its model, gateway URL, and approved credential domains.
- `kit/bifrost.yaml` declares service `bifrost`, port 8080 access, and `x-bf-vk` injection.
- The proxy normalizes `host.docker.internal` to **`localhost`**. Both names must
  be covered by the kit and bindings; the missing `localhost` mapping caused the 401.
- The startup hook configures Claude's header with a placeholder. The proxy
  injects the real virtual key from the host store; Bifrost holds the Anthropic key.
- AIG policies still apply. `sbx policy log` should show `localhost:8080` via `forward`.

## Verify

In Claude, inspect the placeholder:

```text
! echo $BIFROST_VIRTUAL_KEY
```

Expect a nonempty sentinel, not the real virtual key. Ask Claude to say
`bifrost works`, then list the directory using its shell tool. Confirm successful
Anthropic requests in Bifrost's logs.

## Stop the gateway

```bash
docker compose down
```

Configuration persists in `bifrost-local-data`. Local HTTP is used for this demo.
Empty pricing catalogs avoid startup downloads; cost reporting is unavailable.
