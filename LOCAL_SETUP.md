# Khmer Udam ET — Local VS Code Setup

ឯកសារនេះណែនាំការរត់ project ក្នុង VS Code ដោយប្រើ React 19, Vite, Express, tRPC, Drizzle ORM និង MySQL/TiDB។ កុំដាក់ secret ពិតនៅក្នុង Git ឬផ្ញើក្នុង chat។

## 1. តម្រូវការជាមុន

ត្រូវដំឡើង៖

- Node.js 20 ឬថ្មីជាងនេះ (project ត្រូវបានសាកល្បងជាមួយ Node.js 22)
- pnpm 10 (`corepack enable` បន្ទាប់មក `corepack prepare pnpm@10.4.1 --activate`)
- MySQL 8+, MariaDB ឬ TiDB Cloud
- VS Code

បើប្រើ Windows អាចប្រើ MySQL Installer ឬ Docker Desktop។ បើប្រើ macOS អាចប្រើ Homebrew។

## 2. បើក source code

```bash
git clone https://github.com/udomet32-create/khmer-udam-et.git
cd khmer-udam-et
pnpm install
```

បើអ្នកប្រើ ZIP ដែលបានផ្ដល់រួច សូម unzip ហើយបើក folder ខាងក្នុងដែលមាន `package.json` ជា VS Code workspace។

## 3. បង្កើត database

### MySQL local

ចូល MySQL ជា root ហើយរត់៖

```sql
CREATE DATABASE khmer_udam_et CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'khmer_user'@'localhost' IDENTIFIED BY 'ជំនួសដោយពាក្យសម្ងាត់របស់អ្នក';
GRANT ALL PRIVILEGES ON khmer_udam_et.* TO 'khmer_user'@'localhost';
FLUSH PRIVILEGES;
```

បើប្រើ `127.0.0.1` ក្នុង connection string ហើយ MySQL user ត្រូវបានបង្កើតសម្រាប់ `localhost` មិនអាច login បាន សូមបង្កើត user មួយទៀតសម្រាប់ `127.0.0.1` ឬប្រើ host ដូចគ្នាទាំងពីរ។

### TiDB Cloud

បង្កើត database ក្នុង TiDB Cloud ហើយ copy MySQL connection string ពី Connect។ ប្រើ TLS options ដែល TiDB ផ្ដល់ឱ្យ។ ជាទូទៅវាមានទម្រង់៖

```text
mysql://USER:PASSWORD@HOST:4000/DATABASE?ssl={"rejectUnauthorized":true}
```

កុំបញ្ចូល connection string ពិតទៅក្នុង GitHub។

## 4. Environment variables

បង្កើតឯកសារ `.env` នៅ root project ដោយយកឈ្មោះ variables ខាងក្រោម។ `.env` ត្រូវបាន ignore ដោយ Git ហើយមិនត្រូវ commit។

```dotenv
NODE_ENV=development
PORT=3000
DATABASE_URL=mysql://khmer_user:YOUR_PASSWORD@127.0.0.1:3306/khmer_udam_et
JWT_SECRET=GENERATE_A_LONG_RANDOM_SECRET_32_CHARS_OR_MORE

# Manus OAuth — ត្រូវការសម្រាប់ Manus login និង callback
VITE_APP_ID=YOUR_MANUS_APP_ID
OAUTH_SERVER_URL=https://api.manus.im
OWNER_OPEN_ID=YOUR_OWNER_OPEN_ID

# Manus Forge — ត្រូវការសម្រាប់ upload product/poster/video/QR និង media proxy
BUILT_IN_FORGE_API_URL=YOUR_FORGE_API_URL
BUILT_IN_FORGE_API_KEY=YOUR_FORGE_API_KEY

# Map component (frontend)
VITE_FRONTEND_FORGE_API_URL=https://forge.butterfly-effect.dev
VITE_FRONTEND_FORGE_API_KEY=YOUR_FRONTEND_FORGE_API_KEY

# Optional analytics; អាចទុកទទេ
# VITE_ANALYTICS_ENDPOINT=
# VITE_ANALYTICS_WEBSITE_ID=
```

បង្កើត `JWT_SECRET` ដោយប្រើ៖

```bash
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

Project ផ្ទុក `.env` ដោយ `dotenv/config` នៅពេល server ចាប់ផ្ដើម។ ឈ្មោះ environment variables ខាងលើត្រូវតែសរសេរត្រឹមត្រូវ។

### អំពី Manus Forge និង local mode

មុខងារ upload រូបភាព, poster, វីដេអូ, QR និង signed media URLs ប្រើ Manus Forge/S3 adapter ដែលមានស្រាប់ក្នុង project។ បើអ្នកមិនមាន `BUILT_IN_FORGE_API_URL` និង `BUILT_IN_FORGE_API_KEY` សម្រាប់ local environment ទេ storefront ដែលប្រើទិន្នន័យមាន URL រួចអាចបើកបាន ប៉ុន្តែ upload media ថ្មី និង map proxy នឹងមិនដំណើរការ។ កុំយក production key ដែលបានបង្ហាញក្នុង log ឬ chat ទៅ commit។

## 5. Apply Drizzle schema/migrations

បន្ទាប់ពី `.env` រួច៖

```bash
pnpm db:push
```

Script នេះរត់ `drizzle-kit generate` ហើយបន្ទាប់មក `drizzle-kit migrate`។ វានឹងបង្កើត/អនុវត្តតារាង `users`, `products` និង `siteSettings` តាម schema បច្ចុប្បន្ន។ ប្រសិនបើ database មាន tables រួច សូម backup មុនរត់ migration។

ពិនិត្យ schema ដោយ៖

```bash
pnpm check
```

កុំរត់ `drizzle-kit push` លើ production database ដោយគ្មាន backup និង review SQL ជាមុន។

## 6. Google Sign-In សម្រាប់ Admin

Google login ក្នុង source code ប្រើ Google Identity Services ហើយ backend verify ID token ជាមួយ Google Client ID ដែលកំណត់ក្នុង `shared/const.ts`។ Client ID ដែល project បានប្រើគឺ៖

```text
1050266900219-ee9kk1peliamj89i8ir40fnk3brugpoj.apps.googleusercontent.com
```

ក្នុង Google Cloud Console៖

1. បើក project **Khmer Udam ET Login**។
2. ចូល **APIs & Services → Credentials** ហើយពិនិត្យ OAuth Client ID ជាប្រភេទ Web application។
3. បន្ថែម local origin ទៅ **Authorized JavaScript origins**៖ `http://localhost:3000` និង `http://127.0.0.1:3000`។
4. បើប្រើ port ផ្សេង ត្រូវបន្ថែម origin នោះផង។
5. កំណត់ test user ជា `dom707768@gmail.com` ឬកែ `GOOGLE_ADMIN_EMAIL` ក្នុង `shared/const.ts` ប្រសិនបើចង់ប្រើ admin email ផ្សេង។

ចំណាំ៖ Google OAuth មិនអនុញ្ញាតឱ្យដាក់ wildcard ដូចជា `http://localhost:*` ទេ។ ត្រូវបញ្ចូល origin ជាក់លាក់។ កុំផ្ញើ Gmail password; ប្រព័ន្ធត្រូវប្រើ OAuth credential ប៉ុណ្ណោះ។

## 7. Telegram order notifications

Telegram integration ប្រើ server-side environment variables ខាងក្រោម៖

```dotenv
TELEGRAM_BOT_TOKEN=YOUR_BOT_TOKEN
TELEGRAM_CHAT_ID=YOUR_CHAT_ID
```

នៅក្នុង source version បច្ចុប្បន្ន bot token និង chat ID ត្រូវបានគ្រប់គ្រងជាសម្ងាត់ដោយ project environment។ សម្រាប់ standalone VS Code សូមប្រើ token ថ្មីពី `@BotFather` ប្រសិនបើ token ចាស់ត្រូវបានបង្ហោះជាសាធារណៈ ឬសង្ស័យថាត្រូវបានបែកធ្លាយ។ កុំដាក់ token ក្នុង frontend code។

ដើម្បីយក Chat ID៖

1. បង្កើត bot ជាមួយ `@BotFather` និង copy token។
2. Add bot ទៅ group/channel ហើយផ្ដល់សិទ្ធិឱ្យ bot ផ្ញើសារ។
3. ផ្ញើសារមួយទៅ group/channel។
4. ប្រើ `getUpdates` ជាមួយ token ដើម្បីស្វែងរក `chat.id` ឬប្រើ bot ដែលបង្ហាញ chat ID។
5. ដាក់ token និង ID ក្នុង `.env` ហើយ restart dev server។

Telegram network អាចត្រូវបាន block នៅបណ្តាញខ្លះ។ បើ notification មិនទៅ សូមពិនិត្យ server log និង token/chat ID មុនពិនិត្យ frontend។

## 8. Run និង test

```bash
pnpm dev
```

បើក `http://localhost:3000`។ បើ port 3000 កំពុងប្រើ server នឹងស្វែងរក port ទំនេរបន្ទាប់ដោយស្វ័យប្រវត្តិ។

សម្រាប់ validation៖

```bash
pnpm check
pnpm test
pnpm build
```

## 9. Admin permissions

Google Sign-In ត្រូវតែផ្គូផ្គង email ដែលបានអនុញ្ញាត។ បន្ទាប់ពី user ចូលលើកដំបូង ប្រសិនបើត្រូវការកំណត់ role ក្នុង database អ្នកអាចប្រើ៖

```sql
UPDATE users SET role = 'admin' WHERE email = 'your-admin@example.com';
```

ធ្វើបែបនេះតែសម្រាប់ email ដែលអ្នកទុកចិត្ត និងបន្ទាប់ពី backup database។

## Troubleshooting ខ្លីៗ

| បញ្ហា | ដំណោះស្រាយ |
|---|---|
| `ECONNREFUSED` ទៅ MySQL | ពិនិត្យ MySQL service, host, port, username និង password ក្នុង `DATABASE_URL` |
| `Access denied for user` | ពិនិត្យ user host (`localhost` vs `127.0.0.1`) និង GRANT |
| Upload រូបភាពបរាជ័យ | ត្រូវមាន Forge URL និង API key ត្រឹមត្រូវ ហើយ restart server |
| Google `origin_mismatch` | បន្ថែម `http://localhost:3000` ទៅ Authorized JavaScript origins |
| Google account មិនអាចចូល Admin | email ត្រូវតែស្ថិតក្នុង allowlist នៅ `shared/const.ts` និង user ត្រូវមាន role admin |
| Telegram មិនទទួល order | ពិនិត្យ bot token, chat ID, bot membership/permissions និង network access |
| `pnpm db:push` បរាជ័យ | ពិនិត្យ `.env` ត្រូវនៅ root folder ជាមួយ `package.json` ហើយ database ត្រូវមានសិទ្ធិ create/alter tables |
