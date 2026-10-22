# Way Out

Mobile app where people and businesses can send and receive deliveries, and find and book
professionals: delivery services, personal caregivers, personal chefs, private tutors and
housekeepers. Personal and business/organization accounts have their own sign-up forms.

- `backend/` Node.js + Express + PostgreSQL API
- `mobile/` React Native (Expo) app for Android and iPhone

## Status

First working version. What has been checked so far:

- The database schema was applied to a real PostgreSQL 16 server, twice, with no errors.
- The main new queries (professional search, inbox, bookings, deliveries, live tracking, ratings)
  were run against sample data and returned the expected rows.
- All backend and mobile files were checked for syntax errors.

What has **not** been done: the API server and the mobile app have not been started end to end
(the build environment could not download packages), and there are no automated tests. Expect to
fix small issues the first time you run it.

## Features and where they live

| # | Feature | Where |
|---|---------|-------|
| 1 | Personal account | `POST /auth/register/individual` |
| 2 | Business or organization account | `POST /auth/register/business` |
| 3 | Links to view profiles | `GET /profiles/:slug`, link format `wayout://p/<code>` |
| 4 | Call enable | Account screen switch (`call_enabled`); Call buttons use the phone dialer |
| 5 | Mobile payment | `backend/src/services/payments.js` (mock adapter until you connect a provider) |
| 6 | Price per distance | `backend/src/services/pricing.js`, rates in `.env` |
| 7 | Personal send and receive | `mode` = `send` or `receive` on a delivery |
| 8 | Referral links for businesses only | Code created for business accounts; individuals are rejected if they send one |
| 9 | Security verifications | Email and phone must both be verified before any feature works |
| 10 | Two-factor authentication | Authenticator app (TOTP), `/auth/2fa/*` |
| 11 | Phone code verification | 6-digit SMS code at sign-up |
| 12 | Email verification | 6-digit email code at sign-up |
| 13 | Fingerprint | Fingerprint login, `/auth/biometric/*` |
| 14 | Businesses buy transport slots | `POST /business/slots`, Business tab |
| 15 | Customer ratings | After a delivery or a service; shown on profiles and in search |
| 16 | Profile categories | `categories` table, chosen at sign-up |
| 17 | Direct calls from customer | Call button on profiles, chats, deliveries and bookings |
| 18 | Direct online messages | Messages tab, `/messages/*` (new messages arrive by checking every 4 seconds) |
| 19 | Live map for navigating | Delivery tracking screen: courier position on a map, "Open navigation" opens Google Maps |
| 20 | Professions at sign-up | Delivery service, Personal caregiver, Personal chef, Private tutor, Housekeeper; set at sign-up and under Account |

## How the professions work

- At sign-up (personal or business) a person can tick the professions they offer. Caregiver, chef,
  tutor and housekeeper need a rate (per hour or per visit). Delivery has no rate because it is
  priced by distance.
- Customers use **Find a professional** (Home screen) to search by profession, city or name, then
  open a profile to call, message or send a request.
- A request goes to that one professional. They accept or decline. After they accept, the customer
  pays by mobile money, the professional does the job and marks it completed, and the customer rates it.
- To accept open deliveries, a personal account needs the "Delivery service" profession; a business
  needs an active transport slot with free capacity.
- Professions live in the `professions` table. To add another one (for example "Nurse" or
  "Driver"), insert a row; the app picks it up with no code change.

## Run the backend

Requirements: Node.js 18+, PostgreSQL 13+.

```bash
cd backend
cp .env.example .env        # then edit DATABASE_URL and JWT_SECRET
npm install
npm run db:init             # creates tables, professions, categories and transport types
npm run dev
```

In development, verification codes are printed in the server log instead of being sent.
Check it works: open `http://localhost:4000/health`.

## Run the mobile app

```bash
cd mobile
npm install
EXPO_PUBLIC_API_URL=http://YOUR-COMPUTER-IP:4000 npx expo start
```

Scan the QR code with the Expo Go app. Phone and computer must be on the same Wi-Fi. On an
Android emulator the default `http://10.0.2.2:4000` already points to your computer.

### Maps

The live map uses `react-native-maps`. On iPhone it uses Apple Maps with no setup. For Android
release builds, create a Google Maps API key and put it in `mobile/app.json` in place of
`YOUR_GOOGLE_MAPS_API_KEY`. Fingerprint login and the live map may need a development build
(`npx expo run:android`) if Expo Go does not allow a feature.

## Privacy choices already made

- A profile link shows name, services, ratings and reviews to anyone. The phone number is shown
  only to logged-in, verified accounts, and only if the owner has calls turned on.
- A courier's live position is shared only with the customer, only while the delivery is active,
  and only while the courier has the map screen open.
- Messages can only be sent between fully verified accounts.

## Connecting real services

- SMS: replace `sendSms` in `backend/src/services/notify.js`.
- Email: replace `sendEmail` in the same file.
- Mobile money: add an adapter in `backend/src/services/payments.js` that implements `charge()`
  for your provider, then set `PAYMENT_PROVIDER` in `.env`. Real providers usually confirm
  payments through a callback, so you will also need a webhook route that marks the payment as
  succeeded.

## Known gaps (worth deciding before launch)

- **Provider payouts are not built.** Customers pay through the app, but there is no step that
  pays the courier, caregiver, chef, tutor or housekeeper, no platform fee, and no refunds.
- **No block or report button in chat.** Any verified account can message any other verified account.
- **Caregivers and tutors often work with children or elderly people.** The app verifies phone
  and email only. Consider ID checks, references or background checks before listing them.
- Live location works only while the map screen is open (no background tracking).
- The in-app map shows positions; turn-by-turn directions open in Google Maps. Drawing the road
  route inside the app needs a Google Directions API key.
- Messages are checked every 4 seconds rather than pushed instantly; there are no push notifications.
- Distance is a straight line, not a driving route.
- Addresses are turned into coordinates by the phone's built-in geocoder, which can be unreliable
  for some local addresses. A map pin picker is a good next step.
- Service bookings are typed as a date like `2026-11-02 14:30`; a date picker would be nicer.
- No password reset, admin panel or automated tests yet.
