# Local License Server samples

This repository contains sample Local License Server (LLS) deployments and a Node.js walkthrough script for activating an LLS instance and importing an entitlement.

## Contents

- `simple/` - the most basic LLS setup with a PostgreSQL database.
- `ssl/` - the simple setup with HTTPS enabled directly in ASP.NET Core.
- `haproxy-ssl/` - the simple setup behind HAProxy with HTTPS termination.
- `nginx-ssl/` - the simple setup behind nginx with HTTPS termination.
- `high-availability/` - a high-availability LLS setup using HAProxy, Redis, and multiple LLS instances.
- `lls_activation_walkthrough.js` - an interactive script that activates LLS and imports an entitlement.

## Prerequisites

- Docker and Docker Compose
- Node.js 18 or newer for the activation walkthrough script
- Zentitle2 Management API credentials
- LLS activation code
- Entitlement ID to import

## Simple setup

The simple sample starts one LLS container and one PostgreSQL database.

```bash
cd simple
docker compose up -d
```

LLS is exposed at:

```text
http://localhost:6501
```

The sample uses these default LLS credentials:

```text
Username: admin
Password: admin
```

PostgreSQL is exposed on host port `5433`.

To stop the sample:

```bash
docker compose down
```

## SSL setup

The SSL sample starts one LLS container and one PostgreSQL database. A Compose init service generates a self-signed localhost certificate into a Docker volume, and LLS uses ASP.NET Core environment variables to serve HTTPS directly. The generated certificate is also added to the LLS container trust store so the server-side UI can call its own HTTPS API.

```bash
cd ssl
docker compose up -d
```

LLS is exposed at:

```text
https://localhost:6501
```

The certificate is self-signed, so browsers and API clients will require an exception or a trusted replacement certificate.

To stop the sample:

```bash
docker compose down
```

## HAProxy SSL setup

The HAProxy SSL sample starts one LLS container, one PostgreSQL database, and HAProxy. A Compose init service generates a self-signed localhost certificate into a Docker volume. HAProxy terminates browser-facing HTTPS and proxies to LLS over HTTPS on the internal Docker network.

```bash
cd haproxy-ssl
docker compose up -d
```

Access the LLS API and UI through HAProxy:

```text
https://localhost:6501
```

The HAProxy configuration:

- terminates HTTPS with the generated certificate
- proxies to the LLS backend over HTTPS with the same generated certificate
- forwards `X-Forwarded-*` headers to LLS
- performs readiness checks against `/health/ready`
- keeps upgraded WebSocket traffic open for the server-side UI

The certificate is self-signed, so browsers and API clients will require an exception or a trusted replacement certificate.

To stop the sample:

```bash
docker compose down
```

## nginx SSL setup

The nginx SSL sample starts one LLS container, one PostgreSQL database, and nginx. A Compose init service generates a self-signed localhost certificate into a Docker volume. nginx terminates browser-facing HTTPS and proxies to LLS over HTTPS on the internal Docker network.

```bash
cd nginx-ssl
docker compose up -d
```

Access the LLS API and UI through nginx:

```text
https://localhost:6501
```

The nginx configuration:

- terminates HTTPS with the generated certificate
- proxies to the LLS backend over HTTPS with the same generated certificate
- forwards `X-Forwarded-*` headers to LLS
- supports WebSocket upgrades for the server-side UI
- keeps long-lived UI connections open while idle

The certificate is self-signed, so browsers and API clients will require an exception or a trusted replacement certificate.

To stop the sample:

```bash
docker compose down
```

## High-availability setup

The high-availability sample starts:

- PostgreSQL for LLS storage
- Redis for LLS high-availability coordination
- three LLS instances
- HAProxy on port `6501`

```bash
cd high-availability
docker compose up -d
```

Access the LLS API and UI through HAProxy:

```text
http://localhost:6501
```

The HAProxy configuration:

- load-balances API traffic across the LLS instances
- keeps `/` and `/ui` traffic sticky for the server-side UI
- performs readiness checks against `/health/ready`
- limits concurrent client connections for the sample stack

The sample uses these default LLS credentials:

```text
Username: admin
Password: admin
```

To stop the sample:

```bash
docker compose down
```

## Activation and entitlement import walkthrough

Run the walkthrough after starting either sample:

```bash
node lls_activation_walkthrough.js
```

The script prompts for:

- Zentitle2 Management API URL
- identity provider URL or full token URL
- tenant ID
- Management API client ID and secret
- LLS URL
- LLS username and password
- LLS activation code, when the server is not already active
- LLS server name, when activation is required
- entitlement ID

The script stores non-secret values in `lls_activation_walkthrough.env` so they can be reused on later runs. Secrets and passwords are not stored. The generated `.env` file is ignored by git.

For a locally running sample, use:

```text
LLS URL: http://localhost:6501 or https://localhost:6501
LLS username: admin
LLS password: admin
```

When the LLS URL uses `https://`, the walkthrough accepts the sample's self-signed LLS certificate. Management API and identity provider requests still use normal certificate validation.

## Notes

These samples are intended for local evaluation and development. Change the default credentials and database passwords before adapting them for shared or production-like environments.
