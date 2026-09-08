# Karigar

Karigar is a location-aware home-services marketplace that connects customers with nearby skilled workers. A customer can describe a repair or service request, choose inspection-first or standard pricing, attach context media, and broadcast the job to workers in the relevant area. Workers can manage availability, review nearby opportunities, fund a lead wallet, and express interest in jobs. Administrators have operational tools for moderation, disputes, advertising, and lifecycle maintenance.

This repository contains the working application in two independently started packages:

- `client/`: React 19 single-page application built with Vite, Tailwind CSS, DaisyUI, Framer Motion, and Axios.
- `server/`: Express 5 API backed by MongoDB, Redis, JWT authentication, and Cloudinary media uploads.

## Features

### Customer experience

- Register and sign in with a customer or worker starting mode.
- Create a service request with a title, description, category, skills, location, and optional address.
- Choose inspection-first pricing or a fixed standard rate-card service.
- Capture browser coordinates for nearby worker matching.
- Add a voice transcript and one problem image or video to the request.
- Enable Rocket Mode for emergency dispatch priority.
- Review worker interest and quotes, select a worker, follow job tracking, and cancel when allowed.
- Review completion evidence, confirm completion, add a rating and review, raise a dispute, or claim the seven-day warranty.

### Worker experience

- Maintain a worker profile with headline, description, categories, languages, experience, service radius, and availability.
- Publish live location so the matching engine can find nearby workers.
- Browse a nearby job feed and view jobs already applied to.
- Express interest with a quote, message, voice transcript, and optional profile boost.
- Mark arrival and completion from the job detail workflow.
- Recharge the wallet through a recorded UPI reference and review transaction history.
- Purchase the 30-day Verified Pro subscription for early access to new leads.

### Admin experience

- View marketplace metrics such as users, active jobs, disputes, revenue, worker availability, blocked wallets, and subscriptions.
- List and inspect users, jobs, and disputes.
- Verify or unverify users, block or unblock accounts, and delete invalid records.
- Assign mediators and resolve disputes with notes.
- Create and activate/deactivate location-aware sponsored ads.
- Run pending job lifecycle maintenance manually.

## Architecture

```text
Browser
  |
  | React SPA on http://localhost:5174
  | /api requests are proxied by Vite
  v
Express API on http://localhost:3000
  |-- MongoDB      users, jobs, applications, disputes, media, ads, wallet transactions
  |-- Redis        blacklisted JWT sessions and logout state
  `-- Cloudinary   uploaded job and completion media
```

The API is mounted at `/api`. The root API health check is `GET /` and returns `{ "connection": "OK" }` when the Express process is responding. The server connects to MongoDB and Redis before it starts listening.

## Repository layout

```text
.
├── client/
│   ├── src/
│   │   ├── controllers/       shared application/session state
│   │   ├── models/            API request modules and display models
│   │   └── views/             layouts, reusable components, and role pages
│   ├── vite.config.js         dev server, port, and API proxy
│   ├── package.json
│   └── package-lock.json
├── server/
│   ├── src/
│   │   ├── config/            MongoDB, Redis, and Cloudinary configuration
│   │   ├── controllers/       auth, job, worker, wallet, media, and admin logic
│   │   ├── middleware/        authentication, role checks, and uploads
│   │   ├── models/            Mongoose schemas
│   │   ├── routes/             version-one route composition
│   │   └── utils/              validation, pricing, matching, wallet, and lifecycle helpers
│   ├── server.js              environment loading, connections, and process bootstrap
│   ├── package.json
│   ├── package-lock.json
│   └── .env                   local secrets; ignored by git
└── readme.md
```

## Technology stack

| Area | Technology |
| --- | --- |
| Frontend | React 19, React Router 7, Vite 7 |
| Styling and interaction | Tailwind CSS 4, DaisyUI, Framer Motion, Lucide React, React Hot Toast |
| Frontend HTTP | Axios with an `/api` base path and bearer-token interceptor |
| Backend | Node.js, Express 5, ES modules |
| Database | MongoDB through Mongoose 9 |
| Authentication | bcrypt password hashing, JWT, HttpOnly cookie, bearer token support |
| Session invalidation | Redis token blacklist |
| File uploads | Multer with Cloudinary storage |
| Geospatial matching | MongoDB `2dsphere` indexes and nearby-worker queries |

## Prerequisites

Install the following before running the application:

- Node.js 20 LTS or another recent Node.js release compatible with Vite 7 and Express 5.
- npm.
- A MongoDB deployment or local MongoDB instance.
- A Redis instance reachable by the server.
- A Cloudinary account if media upload is required.

The server has no database or Redis fallback. It must be able to connect to both services during startup.

## Configuration

Create `server/.env`. This file is ignored and must never contain values committed to source control.

```dotenv
REDIS_HOST=your-redis-host
REDIS_PORT=6379
REDIS_PASS=your-redis-password

DB_KEY=mongodb+srv://user:password@cluster.example.mongodb.net/karigar

SECRET_KEY=replace-with-a-long-random-secret
JWT_EXP=7d
JWT_MAX_AGE=604800000
PORT=3000
NODE_ENV=development

CLOUDINARY_CLOUD_NAME=your-cloud-name
CLOUDINARY_API_KEY=your-api-key
CLOUDINARY_API_SECRET=your-api-secret
```

Variable reference:

| Variable | Required | Used for |
| --- | --- | --- |
| `REDIS_HOST` | Yes | Redis hostname or address |
| `REDIS_PORT` | Yes | Redis port |
| `REDIS_PASS` | Yes | Redis password |
| `DB_KEY` | Yes | MongoDB connection string |
| `SECRET_KEY` | Yes | Signing and verifying JWTs |
| `JWT_EXP` | Yes | JWT expiration accepted by `jsonwebtoken`, for example `7d` |
| `JWT_MAX_AGE` | No | Cookie lifetime in milliseconds; `0` or unset omits `maxAge` |
| `PORT` | No | API port; defaults to `3000` |
| `NODE_ENV` | No | Enables secure cookies when set to `production` |
| `CLOUDINARY_CLOUD_NAME` | Required for uploads | Cloudinary cloud name |
| `CLOUDINARY_API_KEY` | Required for uploads | Cloudinary API key |
| `CLOUDINARY_API_SECRET` | Required for uploads | Cloudinary API secret |

The client supports an optional Vite variable:

```dotenv
# client/.env.local
VITE_API_URL=/api
```

When `VITE_API_URL` is omitted, the client defaults to `/api`. In development, `client/vite.config.js` proxies `/api` to `http://localhost:3000`, so the default works without a second browser origin. For a separately hosted API, set `VITE_API_URL` to that API's `/api` URL and configure the server CORS origin in `server/src/app.js` accordingly.

## Local development

Install dependencies in each package:

```bash
cd server
npm install

cd ../client
npm install
```

Start the API in one terminal:

```bash
cd server
npm run dev
```

This runs `nodemon server.js`, loads `server/.env`, connects to MongoDB and Redis, performs an initial lifecycle pass, and starts the API on port `3000` unless `PORT` is set.

Start the frontend in another terminal:

```bash
cd client
npm run dev
```

Open [http://localhost:5174](http://localhost:5174). The Vite development server uses port `5174` and proxies `/api` requests to the API at `http://localhost:3000`.

Useful production-style frontend commands:

```bash
cd client
npm run build
npm run preview
```

There is currently no production start script, Docker configuration, or root-level workspace script in the repository. Run the two packages independently as shown above.

## Authentication and authorization

1. `POST /api/auth/register` or `POST /api/auth/login` creates a JWT and returns it as `Token` in the response.
2. The API also sets an HttpOnly `Token` cookie.
3. The client stores the returned token in local storage under `karigar_auth_token` and sends it as `Authorization: Bearer <token>` on subsequent Axios requests.
4. The authentication middleware accepts either the cookie or bearer token, checks Redis for a blacklisted token, loads the user, and rejects blocked accounts.
5. Admin routes use the same middleware plus an `admin` role check.

All newly registered accounts are created with both `customer` and `worker` modes available. The active mode controls the main dashboard shown by the client; it does not create a second account.

## Core job workflow

The normal lifecycle is:

```text
broadcasting
    -> worker_selected
    -> in_progress
    -> completed_pending_confirmation
    -> completed
```

Alternative states are `cancelled`, `disputed`, and `warranty_claimed`.

1. A customer creates a job. The server validates the required title, description, location, and at least one skill.
2. The job is stored with GeoJSON coordinates when available and matched against available workers in the search radius.
3. Workers express interest. The server records their quote, message, optional voice transcript, and lead/boost choices.
4. The customer selects a worker. The worker can mark arrival, then mark the work complete and upload after-work evidence.
5. The customer confirms completion, optionally rates the worker, and the seven-day warranty is activated.
6. A customer can raise a dispute or claim an active warranty. Administrators can assign a mediator and resolve the dispute.

The server runs lifecycle maintenance once at startup and every 60 seconds afterward. It closes expired confirmation windows, expires warranties, and cancels abandoned jobs after the configured abandonment period. Administrators can also invoke the same maintenance pass from the admin dashboard.

## Current pricing and marketplace rules

These values are currently defined in `server/src/utils/platform.utils.js`:

| Rule | Current value |
| --- | ---: |
| Inspection fee | 50 |
| Trust and safety fee | 15 |
| Worker lead fee | 20 |
| Worker boost fee | 10 |
| Rocket Mode customer fee | 50 |
| Rocket Mode worker bonus | 30 |
| Rocket Mode platform share | 20 |
| Verified Pro fee | 149 for 30 days |
| Verified Pro early access | 10 seconds |
| Worker wallet credit limit | -200 |
| Warranty period | 7 days |
| Default worker search radius | 5 km |
| Dispute window | 2 hours |
| Automatic abandonment refund window | 24 hours |

The standard service rate card currently includes:

| Service code | Service | Category | Price |
| --- | --- | --- | ---: |
| `fan-installation` | Fan Installation | Electrical | 150 |
| `switchboard-repair` | Switchboard Repair | Electrical | 120 |
| `tap-replacement` | Tap Replacement | Plumbing | 180 |
| `pipe-leak-fix` | Pipe Leak Fix | Plumbing | 220 |
| `ac-service-basic` | AC Service (Basic) | Appliance | 499 |
| `deep-cleaning-room` | Deep Cleaning (1 Room) | Cleaning | 299 |
| `door-lock-repair` | Door Lock Repair | Carpentry | 199 |

The pricing model combines the selected standard rate or inspection fee with the trust and safety fee, optional Rocket Mode fee, final quote, and redeemed coins. The API returns the full breakdown from `GET /api/job/rate-card` and job detail responses.

## API reference

The following routes are mounted under `/api`. Unless marked public, they require authentication. Request and response bodies are JSON unless the route is a media upload.

### Public routes

| Method | Route | Purpose |
| --- | --- | --- |
| `POST` | `/auth/register` | Create an account and start a session |
| `POST` | `/auth/login` | Authenticate an existing account |
| `GET` | `/job/rate-card` | Return platform fees and standard service prices |

### Authentication and profile

| Method | Route | Purpose |
| --- | --- | --- |
| `POST` | `/auth/logout` | Blacklist the current token in Redis and clear the cookie |
| `GET` | `/auth/me` | Return the current public user state |
| `PATCH` | `/auth/mode` | Switch between `customer` and `worker` mode |
| `PATCH` | `/auth/profile` | Update identity, languages, skills, UPI, and worker profile fields |
| `PATCH` | `/auth/location` | Update GeoJSON coordinates and/or location text |

### Jobs

| Method | Route | Purpose |
| --- | --- | --- |
| `GET` | `/job/my/list` | List jobs related to the current customer or worker |
| `POST` | `/job/create` | Create and broadcast a job |
| `GET` | `/job/:jobId` | View a job and populated participants |
| `GET` | `/job/:jobId/matches` | View worker matches/applications |
| `POST` | `/job/:jobId/select` | Select a worker |
| `PATCH` | `/job/:jobId/cancel` | Cancel a job when the current state allows it |
| `PATCH` | `/job/:jobId/arrived` | Mark the selected worker as arrived |
| `PATCH` | `/job/:jobId/complete` | Submit completion details and optional after-work media |
| `PATCH` | `/job/:jobId/confirm` | Confirm completion, rating, and review |
| `PATCH` | `/job/:jobId/dispute` | Raise a dispute |
| `PATCH` | `/job/:jobId/warranty-claim` | Claim an active warranty |
| `GET` | `/job/:jobId/tracking` | Return tracking, warranty, safety, recommendations, and a nearby ad |

### Worker and wallet

| Method | Route | Purpose |
| --- | --- | --- |
| `GET` | `/worker/feed` | Return nearby broadcasts and the worker's applications |
| `PATCH` | `/worker/availability` | Update availability, radius, and location data |
| `PATCH` | `/worker/profile` | Update worker-facing profile details |
| `POST` | `/worker/jobs/:jobId/interested` | Express interest and submit a quote |
| `POST` | `/worker/subscription/verified-pro` | Purchase or activate Verified Pro |
| `GET` | `/wallet` | Return wallet, subscription, and coin summary |
| `GET` | `/wallet/transactions` | List wallet transactions |
| `POST` | `/wallet/recharge` | Record a UPI wallet recharge |

### Media

| Method | Route | Purpose |
| --- | --- | --- |
| `POST` | `/media/upload/:jobId` | Upload one multipart file for `customer_context`, `before_work`, or `after_work` |
| `GET` | `/media/list/:jobId` | List media visible to the current job participant |

For uploads, send `multipart/form-data` with a `file` field and a `stage` value. The server stores the file in Cloudinary and saves the resulting media metadata in MongoDB.

### Admin

| Method | Route | Purpose |
| --- | --- | --- |
| `GET` | `/admin/overview` | Marketplace metrics and operational counts |
| `GET` | `/admin/list-users` | List users |
| `GET` | `/admin/get-user-details/:userId` | Inspect one user |
| `PATCH` | `/admin/verify-user/:userId` | Verify a user |
| `PATCH` | `/admin/unverify-user/:userId` | Remove verification |
| `PATCH` | `/admin/block-user/:userId` | Block a user |
| `PATCH` | `/admin/unblock-user/:userId` | Unblock a user |
| `DELETE` | `/admin/delete-user/:userId` | Delete a user |
| `GET` | `/admin/list-jobs` | List all jobs |
| `GET` | `/admin/get-job-details/:jobId` | Inspect one job |
| `DELETE` | `/admin/delete-job/:jobId` | Delete a job |
| `GET` | `/admin/list-disputes` | List disputes |
| `GET` | `/admin/get-dispute-details/:disputeId` | Inspect one dispute |
| `PATCH` | `/admin/assign-mediator/:disputeId` | Assign a mediator |
| `PATCH` | `/admin/disputes/:disputeId/resolve` | Resolve a dispute |
| `DELETE` | `/admin/delete-dispute/:disputeId` | Delete a dispute |
| `GET` | `/admin/get-all-mediator` | List mediator accounts |
| `GET` | `/admin/get-all-employer` | List customer accounts |
| `GET` | `/admin/get-all-labourers` | List worker accounts |
| `GET` | `/admin/ads` | List sponsored ads |
| `POST` | `/admin/ads` | Create a sponsored ad |
| `PATCH` | `/admin/ads/:adId/toggle` | Toggle an ad's active state |
| `POST` | `/admin/maintenance/run` | Run pending job lifecycle maintenance |

## Frontend routes

Public routes:

- `/` landing page
- `/login` sign in
- `/register` account creation

Protected routes:

- `/app/customer/dashboard`
- `/app/customer/new-job`
- `/app/customer/jobs/:jobId`
- `/app/worker/feed`
- `/app/worker/profile`
- `/app/worker/wallet`
- `/app/worker/jobs/:jobId`
- `/app/profile`
- `/app/admin/overview`
- `/app/admin/users`
- `/app/admin/users/:userId`
- `/app/admin/jobs`
- `/app/admin/jobs/:jobId`
- `/app/admin/disputes`
- `/app/admin/disputes/:disputeId`
- `/app/admin/ads`

The `/app` entry route redirects based on the authenticated user's role and active mode. Unauthenticated users are sent to `/login` and admin users are sent to the admin overview.

## Data model overview

The server uses these main Mongoose models:

- `User`: identity, role, active mode, location, worker profile, wallet, subscription, ratings, and verification state.
- `Job`: request details, GeoJSON location, applications, selected worker, pricing, lifecycle timestamps, warranty, disputes, safety metadata, recommendations, and ad snapshot.
- `WalletTransaction`: wallet credits/debits, lead fees, boosts, refunds, and recharge references.
- `Media`: Cloudinary URL, media type, job, uploader, workflow stage, and caption.
- `Dispute`: job, reporter, worker, mediator, issue, evidence, status, and resolution notes.
- `Ad`: active sponsored content, category, target location, radius, schedule, priority, and CTA.

GeoJSON fields are stored as `[longitude, latitude]` and indexed with MongoDB `2dsphere` indexes for matching and local advertising.

## Validation and verification

Run the frontend production build:

```bash
cd client
npm run build
```

Run syntax checks for the shell-free JavaScript entry points if needed:

```bash
cd server
node --check server.js
```

The repository currently does not define an automated test suite. The server's `npm test` script is the default placeholder that exits with an error, and the client has no test script. A practical manual smoke test is:

1. Start MongoDB, Redis, and the API.
2. Open the client and register a customer.
3. Create an inspection-first job with a location and skill.
4. Switch to worker mode, set availability, and submit interest.
5. Switch back to customer mode, select the worker, and exercise the arrival/completion flow.
6. Log in with an admin account to inspect the job and run maintenance.

## Troubleshooting

### The API exits during startup

Check that `DB_KEY`, `REDIS_HOST`, `REDIS_PORT`, `REDIS_PASS`, `SECRET_KEY`, and `JWT_EXP` are present. The API waits for both MongoDB and Redis before calling `app.listen`.

### The frontend cannot reach the API

Confirm that the server is listening on port `3000` and the frontend is using the default `/api` base URL. The Vite proxy target is defined in `client/vite.config.js`. If using a separately hosted API, set `VITE_API_URL` and update the server's CORS configuration.

### Media upload fails

Confirm all three Cloudinary variables are configured and that the request is multipart form data with a `file` field. The supported workflow stages are `customer_context`, `before_work`, and `after_work`.

### Location matching returns no workers

The worker must be in worker mode, marked available, and have a usable location. The customer job also needs coordinates or a saved profile location. The default matching radius is 5 km and can be adjusted by the worker profile or request payload.

### Login succeeds but protected requests fail

Check that the browser accepts cookies and that the token stored as `karigar_auth_token` is not stale. Logout blacklists the JWT in Redis, so a blacklisted token must be replaced by logging in again.

## Security notes

- Keep `server/.env` out of source control and rotate any secret that has been exposed.
- Use a strong, unique `SECRET_KEY` in every environment.
- Set `NODE_ENV=production` behind HTTPS so authentication cookies use the `secure` flag.
- Review the CORS origin in `server/src/app.js` before deploying the frontend to a new domain.
- Treat admin credentials and Cloudinary credentials as production secrets.
- The current wallet recharge flow records a UPI reference; it is not a payment gateway integration.

## Development conventions

Frontend API calls are grouped by domain in `client/src/models/`. Server routes delegate to controllers, while Mongoose schemas live in `server/src/models/` and shared business rules live in `server/src/utils/`. Follow those boundaries when adding a feature so route definitions, request logic, and persistence remain easy to locate.
