<p align="center">
  <img width="200" height="200" src="assets/logo.png" alt="Barangay Cupang seal">
</p>

<h3 align="center">e-Serbisyo</h3>

<div align="center">

[![Status](https://img.shields.io/badge/status-archived-lightgrey.svg)]()
[![Platform](https://img.shields.io/badge/platform-web%20%7C%20android-blue.svg)]()
[![License](https://img.shields.io/badge/license-academic-lightgrey.svg)](#license)

</div>

---

<p align="center"> Barangay Cupang information dissemination and certificate issuance, on the web and on Android.
    <br>
</p>

## 📝 Table of Contents

- [About](#about)
- [Repository layout](#layout)
- [Getting Started](#getting_started)
- [Configuration](#configuration)
- [Running the tests](#tests)
- [Usage](#usage)
- [Deployment](#deployment)
- [Built Using](#built_using)
- [Authors](#authors)
- [Acknowledgements](#acknowledgement)
- [License](#license)

## 🧐 About <a name = "about"></a>

e-Serbisyo is a capstone project from the Technological University of the Philippines - Taguig Campus. It is a barangay information dissemination and certificate issuance system built for Barangay Cupang, Muntinlupa City. The project started during the COVID-19 pandemic, when residents could not easily visit the barangay hall for announcements, complaints, and certificate requests.

The system has two clients that share one backend:

- **Admin web application** — used by barangay officials and staff to publish announcements, process certificate orders, respond to complaints and reports, manage missing person and missing item posts, verify residents, and view dashboards per office. Access is restricted by role.
- **Resident Android application** — used by residents to read announcements, request certificates, submit blotter complaints, report missing persons and items, send feedback and inquiries, and receive notifications.

Both clients talk to the Laravel application in `web/`. The web application serves Blade pages for the admin dashboard and a JWT-authenticated JSON API under `/api` for the Android app. Real-time updates use Pusher, uploads go to Cloudinary, certificates are rendered with DomPDF, and notifications are delivered through Firebase Cloud Messaging, email, and SMS.

The project is no longer deployed. The original Heroku instance (`barangay-cupang.herokuapp.com`) has been retired. The code is kept as an academic and portfolio record.

## 🗂️ Repository layout <a name = "layout"></a>

```
.
├── assets/     Logo and icon used by this README
├── web/        Laravel 8 application: admin dashboard, landing page, and JSON API
└── android/    Native Android app (Java) for residents and bikers
```

## 🏁 Getting Started <a name = "getting_started"></a>

These instructions get a copy of the project running on your local machine for development and testing. See [deployment](#deployment) for notes on running it on a live system.

### Prerequisites

```
PHP 8.0 or 8.1 with BCMath, Ctype, Fileinfo, JSON, Mbstring, OpenSSL, PDO, and Tokenizer
Composer 2
Node.js 16+ and npm
MySQL 8 or MariaDB 10.4+
Android Studio with Android SDK 31 and JDK 11
```

The web app also expects accounts on Pusher, Cloudinary, Firebase, and SMSGatewayMe. The admin dashboard and most API endpoints run without them; only real-time updates, image uploads, push notifications, and SMS delivery need the credentials.

### Installing the web application

```bash
cd web
cp .env.example .env
composer install
php artisan key:generate
php artisan jwt:secret
```

Edit `.env` and set the database connection, then create the schema and seed the demo data:

```bash
php artisan migrate --seed
```

Install the front-end dependencies and compile the assets:

```bash
npm install
npm run dev
```

Start the application:

```bash
php artisan serve
```

Open `http://localhost:8000`. The landing page is public. The admin dashboard is at `/login`.

The seeder creates two accounts for every role. For example, `superadmin1@gmail.com` is a Super Admin and `certificationstaff1@gmail.com` is a Certification Staff member. Every seeded account uses the password `12341234`. These accounts are demo data only; do not seed a real deployment with them.

Background work (email, SMS, and push notifications) uses the database queue. Run a worker in a separate terminal:

```bash
php artisan queue:work database --tries=3
```

The scheduled commands that expire old reports, feedback, and pictures run through the scheduler:

```bash
php artisan schedule:work
```

### Installing the Android application

Open the `android/` folder in Android Studio. The project targets Android 12 (API 31) and supports Android 5.0 (API 21) and later.

1. Create a Firebase project and download its `google-services.json` into `android/app/`. The repository contains the original project file; replace it with your own if you connect the app to a different backend.
2. Point the app at your local API in `android/app/src/main/java/com/example/eserbisyo/Constants/Api.java`:

   ```java
   public static final String URL = "http://10.0.2.2:8000/";
   ```

   `10.0.2.2` is the host machine as seen from the Android emulator. Use your machine's LAN address when running on a physical device.
3. Build the debug APK:

   ```bash
   cd android
   ./gradlew assembleDebug
   ```

The build writes `app/build/outputs/apk/debug/eSerbisyo.apk`. On Windows, use `gradlew.bat`.

## ⚙️ Configuration <a name = "configuration"></a>

| Setting | Description |
|---|---|
| `DB_CONNECTION`, `DB_HOST`, `DB_PORT`, `DB_DATABASE`, `DB_USERNAME`, `DB_PASSWORD` | MySQL connection. |
| `JWT_SECRET` | Signing key for API tokens. Generate it with `php artisan jwt:secret`. |
| `PUSHER_APP_ID`, `PUSHER_APP_KEY`, `PUSHER_APP_SECRET`, `PUSHER_APP_CLUSTER` | Pusher credentials for real-time admin notifications. |
| `CLOUDINARY_URL`, `CLOUDINARY_UPLOAD_PRESET`, `CLOUDINARY_PATH` | Cloudinary credentials and upload folder for signatures, photos, and documents. |
| `FCM_KEY` | Firebase server key used to send push notifications to the Android app. |
| `SMS_GATEWAY_API_KEY`, `SMS_GATEWAY_DEVICE_ID` | SMSGatewayMe credentials for SMS notifications. |
| `MAIL_MAILER`, `MAIL_HOST`, `MAIL_PORT`, `MAIL_USERNAME`, `MAIL_PASSWORD`, `MAIL_FROM_ADDRESS` | Outgoing email. |
| `QUEUE_CONNECTION` | Queue driver. Use `database` so queued notifications are processed by a worker. |

The Android app has no build-time configuration beyond `Constants/Api.java` and `google-services.json`.

## 🔧 Running the tests <a name = "tests"></a>

```bash
cd web
php artisan test
```

The repository ships with Laravel's example unit and feature tests. There is no automated coverage of the barangay workflows. Treat this suite as a smoke test, not as verification of the business rules.

## 🎈 Usage <a name = "usage"></a>

### Admin web application

Officials sign in at `/login` and see only the modules allowed by their role:

| Role | Responsibilities |
|---|---|
| Super Admin | Manage users, promote staff, verify residents, review audit logs, and open every dashboard. |
| Information Admin / Staff | Publish announcements, documents, ordinances, projects, employees, and the Android APK download. |
| Certification Admin / Staff | Maintain certificate types and requirements, process certificate orders, assign bikers, print certificates, and record transactions. |
| TaskForce Admin / Staff | Handle complaints, blotter reports, missing person and missing item posts, feedback, and inquiries. |
| Biker | Claim available delivery orders and confirm pickup or delivery. |
| Resident | Use the Android app; the web account only holds the profile and verification data. |

Typical certificate flow:

1. A resident selects a certificate in the Android app and submits the requirements.
2. The Certification Staff reviews the order and approves or denies it.
3. A biker claims the delivery, or the resident chooses pickup.
4. Staff prints the certificate as a PDF and closes the order.

### Resident Android application

Residents register in the app, verify their identity through the barangay, and then:

1. Read announcements and like or comment on them.
2. Request a certificate and upload the required documents.
3. Submit a blotter complaint, a missing person report, or a missing item report.
4. Send feedback and inquiries.
5. Track order status and notifications.

Anonymous reports are supported for feedback and reports. Unverified residents cannot post missing person or missing item reports.

## 🚀 Deployment <a name = "deployment"></a>

The original deployment used Heroku with the `Procfile` in `web/`:

```
web: vendor/bin/heroku-php-apache2 public/
worker: php artisan queue:restart && php artisan queue:work database --tries=3
```

Because the Laravel app now lives in `web/`, deploy that folder as the application root. With the Heroku CLI and a subtree push:

```bash
git subtree push --prefix web heroku main
```

Set the environment variables from the [configuration](#configuration) table as config vars, run `php artisan migrate --force` once, and keep a worker dyno running for the queue. Heroku no longer offers free dynos, so a current deployment would use Render, Railway, Fly.io, or a conventional PHP host.

For the Android app, build a release APK or App Bundle after setting the production API URL and your own `google-services.json`:

```bash
cd android
./gradlew assembleRelease
```

> **Warning:** Do not commit real credentials. The original repository contained a Firebase server key and SMSGatewayMe token in source; they have been replaced with environment variables. Rotate any key that was ever committed to a public repository.

## ⛏️ Built Using <a name = "built_using"></a>

- [Laravel 8](https://laravel.com/) - Web framework and JSON API
- [PHP](https://www.php.net/) - Server language
- [MySQL](https://www.mysql.com/) - Database
- [Blade](https://laravel.com/docs/8.x/blade) + [Bootstrap 4/5](https://getbootstrap.com/) + [SB Admin 4](https://startbootstrap.com/theme/sb-admin-2) - Admin dashboard
- [jQuery](https://jquery.com/) - Dashboard interactions
- [Pusher](https://pusher.com/) - Real-time notifications
- [Cloudinary](https://cloudinary.com/) - Image and document storage
- [DomPDF](https://github.com/dompdf/dompdf) - Certificate PDF rendering
- [tymon/jwt-auth](https://github.com/tymondesigns/jwt-auth) - API authentication
- [SMSGatewayMe](https://smsgateway.me/) - SMS notifications
- [Java](https://www.java.com/) + [Android SDK](https://developer.android.com/) - Resident app
- [Firebase Cloud Messaging](https://firebase.google.com/docs/cloud-messaging) - Push notifications

## ✍️ Authors <a name = "authors"></a>

- [@diazjhozua](https://github.com/diazjhozua) - Idea, web application, and Android application
- [@DiamondShy](https://github.com/DiamondShy) - Capstone teammate

See also the list of [contributors](https://github.com/diazjhozua/e-Serbisyo/contributors) who participated in this project.

## 🎉 Acknowledgements <a name = "acknowledgement"></a>

- Barangay Cupang, Muntinlupa City, for the domain knowledge and for allowing the system to be built around their services.
- The capstone adviser and panel at the Technological University of the Philippines - Taguig Campus for their guidance.
- The README structure follows [The Documentation Compendium](https://github.com/kylelobo/The-Documentation-Compendium) by [@kylelobo](https://github.com/kylelobo).
- Icons in the admin dashboard are from [Flaticon](https://www.flaticon.com/); attribution is recorded in [`web/credits.txt`](web/credits.txt).
- Thanks to the maintainers of Laravel, Pusher, Cloudinary, Firebase, and the other open-source projects this system depends on.

## License <a name = "license"></a>

This project was produced as an academic capstone at the Technological University of the Philippines - Taguig Campus. The source code and documentation are the property of the authors and the university. All rights reserved. See [`LICENSE`](LICENSE) for the full terms.

The repository is shared publicly for portfolio and reference purposes. You may read the code, but you may not copy, redistribute, or use it commercially without written permission from the authors.
