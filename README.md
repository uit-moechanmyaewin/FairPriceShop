# FairPriceShop

FairPriceShop (a.k.a. StoreKing) is a Laravel 10 + Vue 3 omnichannel commerce stack that combines a customer-facing storefront, an admin back office, and an in-store POS experience. It ships with a guided web installer, rich catalog tooling, multi-channel promotions, deep payment gateway coverage, notification pipelines, and delivery logistics in a single codebase.

## Highlights
- Laravel 10 backend with Sanctum API tokens, modular controllers, policies, jobs, and a rich enum catalog in `app/Enums`
- Vue 3 SPA powered by Vite, Vue Router, Vuex, Tailwind CSS, ApexCharts, and a component library in `resources/js`
- Browser-based installer at `/install` to validate requirements, license keys, and bootstrap environment settings
- Global catalog management: categories with adjacency lists, brands, attributes, variations, stock, suppliers, damage tracking, returns, and purchase orders
- POS and online order pipelines with analytics, invoices, credit balance reports, fulfillment states, and delivery boy dashboards
- Multilingual (Laravel localization + runtime language files), multi-currency, tax management, and configurable units
- Comprehensive engagement tools: sliders, benefits, promotions, coupon engine, wishlists, newsletters, push notifications, SMS, email, cookie consent, and analytics widgets
- Payment gateway integrations out-of-the-box: PayPal, Stripe, Razorpay, Paystack, Cashfree, PhonePe, Paytm, Mollie, MercadoPago, bKash, Flutterwave (Rave), Skrill, Twilio/Vonage for OTP flows, and more (see `composer.json`)

## Tech Stack
- PHP 8.1+, Laravel 10, Sanctum, Spatie Permission & Media Library, Adjacency List, Smartisan Settings
- MySQL 8.x (default) or compatible relational database via Laravel's database layer
- Node 18+ (recommended) with Vite 5, Vue 3, Vuex, Tailwind CSS, Bootstrap 5, Swiper, ApexCharts
- Composer for PHP dependency management; npm (or pnpm/yarn) for front-end assets
- Optional: Redis/Queue workers for async notifications, Horizon (if you enable queue drivers)

## Project Structure Overview
- `app/Http/Controllers/Admin` and `Frontend`: RESTful controllers that back the SPA and installer
- `app/Enums`: Centralized enums for order states, payment channels, analytics cards, inputs, discount types, etc.
- `app/Models`: Eloquent models with relationships for catalog, user, order, POS, and configuration domains
- `app/Services`, `app/Libraries`: Integrations for gateways, notifications, and domain services
- `database/seeders`: Opinionated seed data covering company profile, catalog, promotions, POS, orders, and default permissions
- `resources/js`: Vue SPA (layouts, store, modules, router, components, i18n resources)
- `resources/views`: Blade entry points for the SPA shell, email templates, installer screens
- `lang/{locale}`: JSON and PHP translation files for multi-language content
- `routes/api.php`: Versioned API endpoints for admin, storefront, delivery agents, and auth flows

## Prerequisites
- PHP 8.1+ with extensions: OpenSSL, PDO, Mbstring, Tokenizer, cURL, XML, BCMath, GD, Intl, Zip, and `ext-http`
- Composer 2.5+
- Node.js 18 LTS (or newer) + npm 9 (or pnpm/yarn)
- MySQL 8.x (or MariaDB 10.6+) or equivalent database
- Optional but recommended: Redis 6.x for queues, cache, and websockets

## Quick Start
1. Clone or download the project.
2. Copy environment template and generate an app key:
	```bash
	cp .env.example .env
	php artisan key:generate
	```
3. Configure `.env` values:
	- `APP_NAME`, `APP_URL`, `APP_ENV`, `APP_DEBUG`
	- Database: `DB_CONNECTION`, `DB_HOST`, `DB_PORT`, `DB_DATABASE`, `DB_USERNAME`, `DB_PASSWORD`
	- Queue/mail/sms/push/payment credentials (see **Integrations** below)
4. Install PHP dependencies:
	```bash
	composer install
	```
5. Install front-end dependencies:
	```bash
	npm install
	```
6. Link the storage directory (required for media uploads):
	```bash
	php artisan storage:link
	```
7. Run database migrations with seed data (creates demo catalog, roles, admin user, etc.):
	```bash
	php artisan migrate --seed
	```
8. (Optional) Start the SPA asset watcher:
	```bash
	npm run dev
	```
9. Serve the application:
	```bash
	php artisan serve
	```
10. Visit `http://localhost:8000/install` to launch the guided installer if the app is not yet marked as installed.

## Default Credentials
Seeders create baseline administrative users and demo data. Review `database/seeders/UserTableSeeder.php` for generated admin emails and reset the credentials via password reset or database update after installation.

## Integrations
- **Payments**: Configure credentials for gateways you plan to use in `.env` (`PAYPAL_*`, `STRIPE_*`, `RAZORPAY_*`, `PAYSTACK_*`, `BKASH_*`, `PHONEPE_*`, `MOLLIE_*`, `CASHFREE_*`, `PAYTM_*`, etc.). The admin UI exposes toggle/config screens under Settings → Payment Gateway.
- **Notifications**: SMTP (`MAIL_*`), SMS providers (`TWILIO_*`, `VONAGE_*`, gateway-specific keys), and push notifications via Firebase (`FIREBASE_*`).
- **Analytics & Social**: Tracker IDs (Google Analytics, Facebook Pixel) and social links managed from the admin settings.
- **Maps & Delivery**: Google Maps keys when using location-based delivery zones.

## Local Development Tips
- Use `php artisan optimize:clear` after modifying config, route, or translation files.
- Sanctum protects API routes; use `sanctum/csrf-cookie` for SPA authentication flows.
- Adjust `config/filesystems.php` if you store media on S3 or other drivers.
- Queue-heavy features (notifications, order processing) expect a running worker: `php artisan queue:work`.
- Vue SPA entry lives in `resources/js/main.js`; update `vite.config.js` if you customize asset paths.

## Testing
- Feature/Unit tests: `php artisan test`
- Static analysis and coding standards: `./vendor/bin/pint`
- Browser smoke testing: ensure the SPA builds with `npm run build`.

## Deployment Checklist
- Set `APP_ENV=production` and `APP_DEBUG=false`
- Run `php artisan optimize` and `php artisan config:cache`
- Build assets: `npm run build`
- Ensure queues, schedulers (`php artisan schedule:work`), and cron entries are configured
- Set correct file permissions for `storage` and `bootstrap/cache`
- Configure SSL, trusted proxies (`App\Http\Middleware\TrustProxies`), and caching (`config/cache.php`)

## GitHub / CI Suggestions
- Keep `.env` secrets out of version control (already covered by `.gitignore`)
- Exclude `node_modules`, `vendor`, `public/build`, compiled caches (already ignored)
- For CI/CD, run `composer install --no-dev --optimize-autoloader` and `npm ci && npm run build`

## Troubleshooting
- Installer blocked: verify PHP extensions (`php -m`) and directory permissions
- Blank SPA: ensure `npm run dev` (for dev) or `npm run build` + `php artisan vite:install` (production) succeeded
- Payment callbacks failing: double-check webhook URLs, ensure queues are running, and inspect `storage/logs/*.log`
- Missing translations: run the language editor in the admin panel or edit files in `lang/{locale}`

## License
This project inherits the original StoreKing licensing terms. Review any proprietary restrictions before redistribution.
