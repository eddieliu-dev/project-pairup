# PairUp frontend

React application built with Create React App, React Router, and Lucide icons. It includes authentication, onboarding, discovery, profiles, invitations, partner chat, session proposals, and settings.

## Development

Start the Express API first using the [root setup instructions](../README.md#run-locally). Then run these commands from `front-end/`:

```bash
npm install
PORT=3001 npm start
```

Open [localhost:3001](http://localhost:3001). The development proxy forwards `/api` requests to `http://localhost:3000`.

The core flows use the backend API. Despite its historical filename, `src/services/mockApi.js` now contains API helpers rather than a local mock-data service. A running API and MongoDB database are required for those flows.

## Commands

| Command | Purpose |
| --- | --- |
| `npm start` | Start the development server |
| `npm test` | Run tests in watch mode |
| `CI=true npm test -- --watchAll=false` | Run tests once |
| `npm run build` | Generate production assets in `build/` |

## API configuration

`src/config/apiBase.js` reads `REACT_APP_API_BASE_URL`. Leave it unset when using the development proxy or Docker's Nginx proxy. For separate frontend and API hosts, set it to the public API origin **before building**:

```bash
REACT_APP_API_BASE_URL=https://your-api-host.example npm run build
```

The value is embedded in the browser bundle and must not contain secrets. Configure the API's `CORS_ORIGIN` to match the frontend origin. Production static hosts also need a fallback to `index.html` for client-side routes.

See the [development guide](../docs/DEVELOPMENT.md) for Docker, environment configuration, and deployment details.
