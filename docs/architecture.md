# OctoFit Tracker architecture

OctoFit Tracker is a three-tier application for a school fitness program. The starter code includes API contracts and intentionally incomplete implementation points so you can use GitHub Copilot agent mode in a test-driven loop.

![Three-tier OctoFit architecture](../.github/images/octofit-architecture.svg)

## Tier boundaries

- **Presentation tier**: React 19, Vite, React Router, Bootstrap, and a shared API client in `octofit-tracker/frontend/src`.
- **Logic tier**: Express and TypeScript routes/services in `octofit-tracker/backend/src` expose `/api/*` behavior on port `8000`.
- **Data tier**: Mongoose models and seed data in `octofit-tracker/backend/src/models` and `src/scripts/seed.ts` persist to MongoDB on port `27017`.

## Ports and environment rules

- MongoDB: `27017`
- Express API: `8000`
- Vite frontend: `5173`
- The browser calls relative `/api` paths. Vite proxies them to `http://localhost:8000`, so the unauthenticated API port stays private in Codespaces. `VITE_API_BASE_URL` can override the base URL.

## API contract

| Capability | Method and path | Expected behavior |
| --- | --- | --- |
| Activity tracking | `GET /api/activities`, `POST /api/activities` | List and create valid activity records. POST accepts `userId`, `teamId`, `type`, and `minutes`; `userId` and `teamId` must be well-formed IDs for an existing user and team; otherwise respond with a deliberate 4xx (never a 500 from a cast error). A created activity stores the submitted `user`, `team`, `type`, and `minutes` plus computed `points` and `occurredAt`. |
| Teams | `GET /api/teams`, `POST /api/teams` | List and create teams with members. Created and listed teams keep the submitted `name` and `mascot`. Reject a team without a name with `400`. |
| Leaderboard | `GET /api/leaderboard` | Return teams ordered by the sum of their activity points, highest first. Each entry has `teamName`, `totalPoints`, and a 1-based `rank`. |
| Workout suggestions | `GET /api/workouts/suggestions/:userEmail` | Recommend workouts from recent activity patterns for the matching seeded user's email address. The first suggestion's `focus` matches that user's goal or recent activity, with a `reason` explaining why. An email with no matching user returns a deliberate 4xx (for example, `404`). |

## Implementation decisions

Record your plan here before editing code.

- Presentation tier decision: Build route-driven React views with a shared fetch helper from `src/api/client.ts`; keep loading, empty, success, and error state handling in each view and the visual components in `src/components`.
- Logic tier decision: Add explicit Express route handlers and service functions. Keep request validation and API error mapping in the service/route layer; return deliberate 4xx responses for invalid input and avoid exposing Mongoose cast errors as 500 responses.
- Data tier decision: Define Mongoose required-field validation and ObjectId references in the model schemas. Seed stable demo records with upserts without clearing existing collections.
- Activity tracking approach: Validate submitted `userId` and `teamId` as well-formed IDs for existing records, then create an activity that records `user`, `team`, `type`, `minutes`, computed `points`, and `occurredAt`. Expose list and create behavior at `/api/activities`.
- Team management approach: Validate a team name in both the schema and API, preserve submitted `name` and `mascot`, and support member ObjectIds in the team model and API.
- Leaderboard approach: Aggregate activity points by team, sort teams from highest to lowest total points, and map the sorted result to entries containing `teamName`, `totalPoints`, and a 1-based `rank`.
- Workout suggestions approach: Find the seeded user by the normalized email route parameter, derive a suggestion focus from recent activity or the user's fitness goal, and return recommendations with `focus`, `reason`, `title`, `minutes`, and `difficulty`. Return a 404 when the email has no matching user.

## Test plan

Record the commands you run and what each failure means.

- Data-tier command: `npm run test:data --prefix octofit-tracker/backend`
  - Interpreting failures: failures show missing Mongoose required validation or references in users, teams, activities, or workouts.
- API command: `npm run test:api --prefix octofit-tracker/backend`
  - Interpreting failures: failures show missing service behavior, route implementation, valid 4xx error handling, leaderboard aggregation/ranking, or user-matched workout logic.
- Frontend command: `npm test --prefix octofit-tracker/frontend`
  - Interpreting failures: failures show missing route rendering or missing loading, empty, success, error, and workout-request behavior.
- Backend build command: `npm run build --prefix octofit-tracker/backend`
  - Interpreting failures: failures indicate Express, Mongoose, or TypeScript type errors in the backend implementation.
- Frontend build command: `npm run build --prefix octofit-tracker/frontend`
  - Interpreting failures: failures indicate React, routing, API-client, or TypeScript/Vite issues in the frontend implementation.

Implementation order:

1. Run the data contract test and implement Mongoose model validation.
2. Seed demo users, teams, activities, and workouts.
3. Run the API contract test and implement activity/team services and routes.
4. Implement leaderboard aggregation and workout matching, then rerun API tests.
5. Implement frontend routes and shared API handling, then rerun frontend tests.
6. Run backend and frontend builds, fixing only errors attributable to the implementation.
