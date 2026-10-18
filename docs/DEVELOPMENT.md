# Development guide

For the project overview and initial setup, see the [README](../README.md).

## Environment

The API loads `.env` from its working directory. Run API commands from `back-end/`, using [`.env.example`](../back-end/.env.example) as a starting point.

| Variable | Purpose |
| --- | --- |
| `MONGODB_URI` | Required development or deployment MongoDB connection string |
| `TEST_MONGODB_URI` | Separate disposable database used by the backend test suite |
| `JWT_SECRET` | Required secret for signing and verifying authentication tokens |
| `PORT` | API port; defaults to `3000` |
| `CORS_ORIGIN` | Allowed browser origin; use `http://localhost:3001` for local development |

For a local MongoDB instance, development and test connection strings can use separate databases:

```env
MONGODB_URI=mongodb://127.0.0.1:27017/pairup
TEST_MONGODB_URI=mongodb://127.0.0.1:27017/pairup_test
JWT_SECRET=replace-with-a-long-random-secret
PORT=3000
CORS_ORIGIN=http://localhost:3001
```

MongoDB must already be running. For Atlas, use your own connection string and configure database access for the machine running the API. Keep real credentials in `.env` or your deployment environment; `.env` is ignored by Git.

The frontend optionally reads `REACT_APP_API_BASE_URL` at build time. Leave it unset for the local development proxy and Docker Compose setup. See [frontend configuration](../front-end/README.md#api-configuration).

## Sample data

A new database starts empty. Register two accounts with overlapping weekly availability to exercise discovery and invitations, using separate browser profiles to keep their sessions separate.

For fixture-based development, the repository also includes a seed script. From `back-end/`:

```bash
node scripts/seedUsersAndFriendRequests.js
```

This script upserts bundled users and friend requests into **`MONGODB_URI`**, replacing matching fixture records. Use a dedicated development database. It prints a development login on completion; the fixture credentials are public and intended for local use.

## Tests

From `back-end/`:

```bash
npm test
```

The Mocha setup requires `TEST_MONGODB_URI`, rejects a URI identical to `MONGODB_URI`, and requires the test URI to contain `test`. **The suite deletes records from test collections**, so use a disposable database with a distinct database name, such as `pairup_test`. Merely passing the URI checks does not establish that a database is safe to clear.

To collect coverage with the same test setup:

```bash
npm run coverage -- --require ./test/setup.js
```

The standalone scoring and explanation tests can run without MongoDB:

```bash
npx --no-install mocha test/matchScoring.test.js test/matchReasons.test.js
```

From `front-end/`:

```bash
CI=true npm test -- --watchAll=false
npm run build
```

The [CI workflow](../.github/workflows/ci.yml) runs backend tests and a frontend production build. Backend CI needs the repository secrets `TEST_MONGODB_URI` and `JWT_SECRET`; its database must be disposable and reachable from the runner. Frontend tests are not currently part of that workflow.

## Docker

Compose starts the API and an Nginx frontend. It connects to the MongoDB instance specified in `back-end/.env`; it does not start MongoDB.

Both Dockerfiles use `npm ci`, which requires lockfiles. Lockfiles are currently ignored by this repository. Generate them locally before building, from the repository root:

```bash
npm --prefix back-end install --package-lock-only --ignore-scripts
npm --prefix front-end install --package-lock-only --ignore-scripts
cp back-end/.env.example back-end/.env
```

Skip the copy if you already configured `.env`. Set `MONGODB_URI` and `JWT_SECRET` before continuing. A database URL using `localhost` points to the API container itself; for MongoDB on a Docker Desktop host, use `host.docker.internal` in the connection string, or use Atlas.

```bash
docker compose up --build
```

Open [localhost:8080](http://localhost:8080). Nginx forwards `/api/` requests to the backend container. Compose overrides the API port to `3000` and its allowed origin to `http://localhost:8080`.

Stop the services with:

```bash
docker compose down
```

The [static health file](http://localhost:3000/health.txt) confirms that Express serves requests; it is not a database readiness check.

## Deployment

The original project README recorded a Railway frontend deployment at [pairup.up.railway.app](https://pairup.up.railway.app). Availability and the current Railway project configuration have not been verified here.

For separate frontend and API hosts:

1. Set the API's `MONGODB_URI`, `JWT_SECRET`, and `CORS_ORIGIN` in the deployment environment.
2. Set the frontend's `REACT_APP_API_BASE_URL` to the API's public origin before building.
3. Configure the frontend host to serve `index.html` for client-side routes.

The included frontend Docker image instead proxies `/api/` to a service named `backend` on port `3000`. That hostname works inside Compose; adapt the Nginx upstream if using a different service layout. Automatic deployments depend on the hosting provider's repository connection and settings.

## Troubleshooting

- **API fails at startup:** check `MONGODB_URI`, database reachability, and that commands run from `back-end/`.
- **No discovery results:** complete onboarding, ensure another profile exists with shared availability, and check whether a pending or accepted invitation already connects the accounts.
- **Browser API requests fail:** confirm the API is on port `3000`, the frontend is on `3001`, and `CORS_ORIGIN` matches. For separately hosted services, check the build-time API URL.
- **Docker fails at `npm ci`:** generate the two service lockfiles before building.
- **Backend tests refuse to start:** configure a distinct disposable `TEST_MONGODB_URI` containing `test`.

## Course materials

The repository retains the original [assignment instructions](../instructions/), [design walkthrough](../UX-DESIGN.md), issue templates, and instructor automation. Dockerfiles, Compose, and the CI workflow supplied the project's deployment and automation evidence. Instructor automation directories include preservation notes; read those before changing their scripts.
