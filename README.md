# AI Blog Platform (MERN + Vite + Tailwind)

A full‑stack blogging platform with AI‑assisted content generation, rich‑text editing, image uploads via ImageKit, role‑aware admin dashboard, and public blog browsing. Built with React (Vite + Tailwind), Node/Express, and MongoDB.

## Features

- Public blog pages with categories, reading view, and comments (approval workflow)
- Admin/user auth with JWT
  - Admin: ENV‑based login
  - Regular users: register + login, create/manage own blogs
- Blog management: create, publish/unpublish (draft), delete
- AI content generation (Google Gemini) from a title/prompt
- Image uploads with automatic optimization (ImageKit)
- Dashboard stats and filtered visibility
- Responsive UI with TailwindCSS and Vite React

## Tech Stack

- Frontend: React 19, Vite 6, React Router 7, TailwindCSS 4, Quill editor, react-hot-toast
- Backend: Node.js, Express 5, Mongoose 8, JWT, Multer, CORS, dotenv
- AI: @google/genai (Gemini 2.5 Flash)
- Media: ImageKit SDK with URL transforms
- Deployment: Vercel configs provided for both client and server

## Monorepo Structure

```
client/   # Vite React app
server/   # Express API (ESM)
```

Key entry points:
- Server: `server/server.js` exposes `/api/*`
- Client: `client/src/main.jsx`, context at `client/src/context/AppContext.jsx`

## Environment Variables

Create a `.env` file in `server/` and a `.env` (or `.env.local`) in `client/`.

Server (`server/.env`):
```
PORT=3000
MONGODB_URI=mongodb://localhost:27017
JWT_SECRET=your_jwt_secret
# Admin login (ENV-based)
ADMIN_EMAIL=admin@example.com
ADMIN_PASSWORD=supersecurepassword
# Google Gemini
GEMINI_API_KEY=your_gemini_api_key
# ImageKit
IMAGEKIT_PUBLIC_KEY=your_imagekit_public_key
IMAGEKIT_PRIVATE_KEY=your_imagekit_private_key
IMAGEKIT_URL_ENDPOINT=https://ik.imagekit.io/your_id
```

Client (`client/.env`):
```
# Base URL of the server API (no trailing slash)
VITE_BASE_URL=http://localhost:3000
```

Notes:
- The database name is appended in code as `/quickblog` (see `server/configs/db.js`). Ensure `MONGODB_URI` omits the db name.
- Authorization header uses the raw token (no Bearer prefix) per `AppContext.jsx`.

## Install & Run (Windows cmd)

1) Install dependencies

```bat
cd client && npm install
cd ..\server && npm install
cd ..
```

2) Start the API server (reads `server/.env`)

```bat
cd server && npm run start
```

Server listens on `http://localhost:%PORT%` (defaults to 3000).

3) In a new terminal, start the client

```bat
cd client && npm run dev
```

Vite prints the local dev URL (usually `http://localhost:5173`). The client calls the server at `VITE_BASE_URL`.

## API Overview

Base URL: `http://localhost:3000`

- Auth/Admin
  - POST `/api/admin/login` → Admin (env creds) or regular user login → `{ token, user }`
  - GET `/api/admin/dashboard` (auth)
  - GET `/api/admin/blogs` (auth) → Admin: all, User: own blogs
  - GET `/api/admin/comments` (auth) → Admin: all, User: on own blogs
  - POST `/api/admin/approve-comment` (auth)
  - POST `/api/admin/delete-comment` (auth)

- Users
  - POST `/api/user/register` → `{ token, user }`

- Blogs
  - GET `/api/blog/all` → public, published only
  - GET `/api/blog/:blogId` → public
  - POST `/api/blog/add` (auth, multipart: image + blog JSON)
    - Fields: `image` (file), `blog` (JSON string: `{ title, subTitle?, description, category, isPublished }`)
  - POST `/api/blog/delete` (auth) → `{ id }`
  - POST `/api/blog/toggle-publish` (auth) → `{ id }` (toggles draft/published)
  - GET `/api/blog/dashboard` (auth) → user‑scoped blogs; admin sees all

- Comments
  - POST `/api/blog/add-comment` → `{ blog, name, content }`
  - POST `/api/blog/comments` → `{ blogId }` returns approved comments

- AI
  - POST `/api/blog/generate` (auth) → `{ prompt }` returns AI text

Auth header: `Authorization: <token>` (no Bearer prefix).

## Image Uploads

- Uses Multer for incoming file, then uploads to ImageKit
- Stored with transformations: quality auto, webp, width 1280
- Returned URL saved in `Blog.image`

## Admin vs User Behavior

- Admin account is defined via `ADMIN_EMAIL`/`ADMIN_PASSWORD`. Logging in with those grants `isAdmin` and dashboard access to all resources.
- Regular users (via `/api/user/register`) can login and only manage their own blogs/comments.

## Development Tips

- Ensure MongoDB is running locally, or update `MONGODB_URI` to a hosted cluster.
- If you change the server port, also update `VITE_BASE_URL` in the client `.env`.
- Token persistence: token is saved in `localStorage` and set on axios defaults at app boot.
- CORS is enabled with default settings.

## Deploying to Vercel

- Client has `client/vercel.json` with SPA rewrites.
- Server has `server/vercel.json` targeting `server.js` using `@vercel/node`.
- Set the same environment variables on Vercel project(s). For the client, set `VITE_BASE_URL` to your deployed API URL.

## Project Scripts

Client:
- `npm run dev` – Vite dev server
- `npm run build` – Production build
- `npm run preview` – Preview built app
- `npm run lint` – ESLint

Server:
- `npm run start` – Start API once
- `npm run server` – Start API in watch mode (nodemon)

## License

This project is provided as‑is without an explicit license. Add a LICENSE file if you plan to open‑source it.
