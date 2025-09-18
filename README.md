# Jessicarosen151 Backend

A production-ready Node.js/Express backend for a legal resources marketplace with roles for Users, Sellers, and Admins. It includes authentication with JWT and OTP email verification, role-based authorization, product/resource management, Stripe payments and webhooks, dashboards, messaging, reviews, newsletters, and admin-managed custom content.

## What problem it solves

- Enables creators/sellers to upload legal resources (documents/PDFs) and sell them.
- Lets buyers discover, filter, and purchase resources securely.
- Handles revenue split (50/50) between platform admin and sellers via Stripe transfers.
- Provides operational tooling for admins (taxonomies, content, promos, dashboards) and for sellers (sales, revenue, catalog management).

## Key features

- Authentication & Authorization: email OTP verification, access/refresh tokens, role gates (USER, ADMIN, SELLER)
- Product/Resource Catalog: rich filters, search, pagination, status lifecycle (draft/pending/approved/rejected)
- Checkout & Payments: Stripe Checkout sessions, webhook handling, refunds/cancellations, automatic revenue split transfers
- Orders: guest or authenticated flow; order lifecycle and stock management
- Reviews & Q&A: public reviews, seller/admin replies to questions
- Messaging: per-resource threads between buyers and sellers/admins
- Media Uploads: multer for file uploads; Cloudinary storage integration
- Content & Taxonomies: admin-managed hero/about/bestseller/legal docs/privacy/terms; countries/states/divisions; practice areas & subtypes
- Dashboards: seller and admin summaries, time series revenue, sales aggregation
- Security: helmet, CORS, XSS clean, Mongo sanitize, rate limiting
- Observability: morgan HTTP logs, winston logger

## Technologies

- Backend:
  - Node.js, Express
  - MongoDB, Mongoose
  - Security: helmet, cors, xss-clean, express-mongo-sanitize, express-rate-limit, csurf (available), joi/express-validator
  - Auth: jsonwebtoken (JWT), bcrypt
  - File Uploads: multer, Cloudinary SDK
  - Payments: stripe
  - Email: nodemailer
  - Realtime (optional scaffolded): socket.io
  - Tooling: dotenv, morgan, winston
  - Testing (dev): jest, supertest, eslint, prettier, nodemon

- Frontend (companion stack used in this project context):
  - Next.js (App Router recommended)
  - Tailwind CSS
  - shadcn/ui component primitives

## Project structure

```text
root
├─ server.js                     # Entry: DB connect + start HTTP server
├─ src/
│  ├─ app.js                    # Express app, middleware, routes, webhook
│  ├─ core/
│  │  ├─ app/appRouter.js       # Global router mounts
│  │  ├─ config/
│  │  │  ├─ config.js           # Env, JWT, email, cloudinary config
│  │  │  └─ logger.js           # Winston logger
│  │  └─ middlewares/
│  │     ├─ authMiddleware.js   # verifyToken, role gates, optionalVerifyToken
│  │     ├─ errorMiddleware.js  # Error handler { status:false, message, error }
│  │     ├─ multer.js           # Multer field configs
│  │     ├─ notFound.js         # 404 handler
│  │     └─ validateRequest.js  # Validation helpers
│  ├─ entities/
│  │  ├─ auth/                  # Auth register/login/refresh/OTP/password
│  │  ├─ auth.guest/            # Guest creation
│  │  ├─ user/                  # Profiles, avatars, PDFs, orders
│  │  ├─ resource/              # Products CRUD, filters, seller scope
│  │  ├─ review/                # Reviews
│  │  ├─ QuesAns/               # Q&A threads
│  │  ├─ message/               # Messaging by resource thread
│  │  ├─ blog/                  # Blog CRUD (admin)
│  │  ├─ country/               # Country/State/Division taxonomy
│  │  ├─ rTypes/                # Resource types taxonomy
│  │  ├─ subPracticeArea/       # Sub practice area taxonomy
│  │  ├─ practiceArea/          # Practice areas (with sub-areas)
│  │  ├─ promoCode/             # Promo codes + apply
│  │  ├─ Seller/
│  │  │  ├─ Application/        # Become seller
│  │  │  ├─ Dashboard/          # Seller analytics
│  │  │  └─ Stripe-onboard/     # Express onboarding + links
│  │  ├─ Payment/               # Checkout + Webhook controllers/services/models
│  │  └─ admin/
│  │     ├─ Dashboard/          # Admin analytics
│  │     └─ custom/             # Hero, About, BestSeller, LegalDoc, Privacy, Terms
│  └─ lib/
│     ├─ cloudinaryUpload.js
│     ├─ confirmMail.js         # OTP email sender
│     ├─ emailTemplates.js
│     ├─ limit.js               # globalLimiter
│     ├─ pagination.js
│     ├─ responseFormate.js     # generateResponse helper
│     └─ sendEmail.js
├─ uploads/                     # Local upload mount (served at /uploads)
└─ ecosystem.config.js          # PM2 config (if used)
```

## API base paths

- Global: `/api` (see `src/core/app/appRouter.js`)
- Auth: `/api/v1/auth`
- Guest: `/api/v1/guest`
- User: `/api/v1/user`
- Resource: `/api/v1/resource`
- Reviews: `/api/v1/reviews`
- Blog: `/api/v1/blog`
- Country/State/Division: `/api/v1/country-state`
- Resource Types: `/api/v1/resource-type`
- Sub Practice Area: `/api/v1/sub-resource-type`
- Practice Areas: `/api/v1/practice-area`
- Promo Codes: `/api/v1/promo-codes`
- Message: `/api/v1/message`
- Newsletter: `/api/v1/newsletter`
- Q&A: `/api/v1/qa`
- Seller Application: `/api/v1/seller`
- Stripe Onboarding: `/api/v1/stripe`
- Payment: `/api/v1/payment` (webhook: `POST /api/v1/payment/webhook`)
- Dashboards: `/api/v1/seller/dashboard`, `/api/v1/admin/dashboard`
- Admin custom content: `/api/v1/admin/custom/{hero|about|bestseller|legal-document|privacy|terms}`

## Core middleware & behaviors

- Security: `helmet`, `cors("*")`, `xss-clean`, `express-mongo-sanitize`
- Parsing: JSON (10MB), `urlencoded`, cookies; Stripe webhook uses `bodyParser.raw` on route
- Rate limiting: `globalLimiter` from `src/lib/limit.js`
- Static assets: `/uploads` serves local `uploads/`
- Errors: 404 via `notFound`; error handler returns `{ status:false, message, error }`
- Responses: use `generateResponse(res, statusCode, status, message, data)` from `src/lib/responseFormate.js`

## Environment variables

Create a `.env` in project root:

```bash
PORT=5000
NODE_ENV=development
MONGO_URI=mongodb+srv://...

# JWT
JWT_SECRET=...
JWT_EXPIRE=1h
ACCESS_TOKEN_SECRET=...
ACCESS_TOKEN_EXPIRES=7d
REFRESH_TOKEN_SECRET=...
REFRESH_TOKEN_EXPIRES=10d
SALT=10

# Email (Nodemailer)
EMAIL_HOST=smtp.example.com
EMAIL_PORT=587
EMAIL_ADDRESS=no-reply@example.com
EMAIL_PASS=your_password
EMAIL_FROM="Your App <no-reply@example.com>"

# Cloudinary
CLOUDINARY_CLOUD_NAME=...
CLOUDINARY_API_KEY=...
CLOUDINARY_API_SECRET=...

# Frontend
FRONTEND_URL=https://lawbie.com/

# Stripe
STRIPE_SECRET_KEY=sk_live_or_test
STRIPE_WEBHOOK_SECRET=whsec_...
```

## Getting started

```bash
# 1) Install
npm install

# 2) Configure env
cp .env.example .env   # if you have an example file, otherwise create manually

# 3) Run in dev
npm run dev

# 4) Production
npm start
```

- Server runs on `PORT` (default 5000) and connects to `MONGO_URI`.
- Webhook endpoint: ensure Stripe dashboard points to `POST /api/v1/payment/webhook` and your server exposes raw body for that route.

## Frontend usage (Next.js + Tailwind + shadcn/ui)

- Consume this API from a Next.js app.
- Use Tailwind for styling and shadcn/ui components.
- Stripe Checkout: redirect users to the `url` returned by `POST /api/v1/payment/create-session`.

## Development notes

- Uses ES modules (`type: module`).
- Webhook route must appear before JSON body parser or must explicitly use `bodyParser.raw` (already handled in `src/app.js`).
- Many read endpoints are public; write endpoints are role-protected. Review route files under `src/entities/**` for details.
- Reviews creation route is currently unauthenticated by business choice.

## Testing

- Jest + Supertest are configured in dev dependencies. Add tests in a `__tests__/` directory or alongside modules.

## License

Proprietary or custom; update this section as needed.
