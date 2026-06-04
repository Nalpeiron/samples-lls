# Local License Server samples

This repository contains sample Local License Server (LLS) deployments and a Node.js walkthrough script for activating an LLS instance and importing an entitlement.

## Contents

- `simple/` - the most basic LLS setup with a PostgreSQL database.
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
LLS URL: http://localhost:6501
LLS username: admin
LLS password: admin
```

## Notes

These samples are intended for local evaluation and development. Change the default credentials and database passwords before adapting them for shared or production-like environments.
