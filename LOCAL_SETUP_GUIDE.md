# Khmer Udam ET — Local VS Code Setup

ឯកសារនេះណែនាំការរត់ `Khmer Udam ET` ពី source code ក្នុង VS Code។ Project ប្រើ React/Vite នៅខាងមុខ, Express/tRPC នៅខាងក្រោយ, Drizzle ORM និង MySQL-compatible database ដូចជា MySQL ឬ TiDB។

## 1. តម្រូវការមុនចាប់ផ្តើម

ត្រូវដំឡើង៖

- Node.js 20 ឬថ្មីជាងនេះ
- pnpm 10 ឬថ្មីជាងនេះ
- MySQL 8+ ឬ TiDB Cloud / TiDB Serverless
- Git (ជាជម្រើស)

បើមិនទាន់មាន pnpm សូមបើក Terminal ក្នុង VS Code ហើយរត់៖

```bash
npm install -g pnpm
```

បន្ទាប់មក extract ZIP ហើយបើក **folder ខាងក្នុងដែលមាន `package.json`** ជា VS Code workspace។

## 2. ដំឡើង packages

```bash
pnpm install
```

កុំបង្កើត project ថ្មីពីក្នុង folder នេះ ព្រោះ source code និង migration files មានរួចហើយ។

## 3. បង្កើត Database

### ជម្រើស A — MySQL ក្នុងកុំព្យូទ័រ

ចូល MySQL ហើយបង្កើត database ទទេ៖

```sql
CREATE DATABASE khmer_udam_et
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;
```

បើប្រើ user ផ្សេង សូមផ្តល់សិទ្ធិឱ្យ user នោះលើ database នេះ។

### ជម្រើស B — TiDB Cloud

បង្កើត cluster/database នៅ TiDB Cloud រួច copy connection string ពី **Connect > General / Node.js**។ ជ្រើស connection string ដែលប្រើ MySQL protocol ហើយរក្សា SSL parameters តាម string ដែល TiDB ផ្តល់ឱ្យ។

## 4. បង្កើតឯកសារ `.env`

នៅ root folder ដែលមាន `package.json` បង្កើត file ឈ្មោះ `.env`។ កុំដាក់ `.env` ក្នុង Git ហើយកុំផ្ញើ file នេះទៅអ្នកដទៃ។ Copy template ខាងក្រោម រួចប្តូរ value ក្នុង `<...>`៖

```dotenv
# Runtime
NODE_ENV=development
PORT=3000

# Database — required
# Local MySQL example:
DATABASE_URL=mysql://root:<MYSQL_PASSWORD>@127.0.0.1:3306/khmer_udam_et
# TiDB example: use the exact connection string supplied by TiDB Cloud.
# Do not keep both DATABASE_URL lines active; use only one.

# Session / Manus OAuth — required for the existing admin login flow
JWT_SECRET=<long-random-secret-at-least-32-characters>
VITE_APP_ID=<manus-app-id>
OAUTH_SERVER_URL=<manus-oauth-server-url>
VITE_OAUTH_PORTAL_URL=<manus-login-portal-url>
OWNER_OPEN_ID=<owner-open-id>
OWNER_NAME=<owner-display-name>

# Manus built-in storage/API — required by the existing Manus-backed media flow
BUILT_IN_FORGE_API_URL=<built-in-forge-api-url>
BUILT_IN_FORGE_API_KEY=<server-side-forge-api-key>
VITE_FRONTEND_FORGE_API_URL=<frontend-forge-api-url>
VITE_FRONTEND_FORGE_API_KEY=<frontend-forge-api-key>

# Telegram order notifications — optional, but required if you want order alerts
TELEGRAM_BOT_TOKEN=<telegram-bot-token>
TELEGRAM_CHAT_ID=<telegram-chat-id>
```

### អត្ថន័យរបស់ variables សំខាន់ៗ

| Variable | តួនាទី | ត្រូវការ |
|---|---|---|
| `DATABASE_URL` | Connection ទៅ MySQL/TiDB | ចាំបាច់ |
| `JWT_SECRET` | ចុះហត្ថលេខា session cookie | ចាំបាច់សម្រាប់ login; ប្រើ secret ថ្មីពេល local |
| `VITE_APP_ID` | App ID របស់ Manus OAuth | ចាំបាច់សម្រាប់ flow ដែលមានស្រាប់ |
| `OAUTH_SERVER_URL` | OAuth backend URL | ចាំបាច់សម្រាប់ login |
| `VITE_OAUTH_PORTAL_URL` | URL ទំព័រ login ដែល frontend បើក | ចាំបាច់សម្រាប់ login |
| `OWNER_OPEN_ID` / `OWNER_NAME` | កំណត់ owner និង admin ដើម | ចាំបាច់សម្រាប់ admin authorization |
| `BUILT_IN_FORGE_API_*` | Storage/API របស់ Manus | ចាំបាច់សម្រាប់ feature ដែលពឹងលើ Manus storage/API |
| `TELEGRAM_BOT_TOKEN` | Token របស់ bot | ត្រូវការសម្រាប់ Telegram notification |
| `TELEGRAM_CHAT_ID` | Chat/channel ទទួល order alert | ត្រូវការសម្រាប់ Telegram notification |
| `PORT` | Port local server | ជាជម្រើស; default គឺ `3000` |

Project បច្ចុប្បន្នមាន Google client ID និង admin email នៅក្នុង `shared/const.ts` រួចហើយ ដូច្នេះមិនមាន `GOOGLE_CLIENT_ID` variable ដាច់ដោយឡែកក្នុង source version នេះទេ។ កុំដាក់ Google client secret ក្នុង frontend ឬ commit ទៅ Git។

> សំខាន់៖ កុំយក Telegram bot token ដែលបានបង្ហាញនៅក្នុង chat ទៅ commit ក្នុង Git។ ប្រសិនបើ token ត្រូវបានចែករំលែកជាសាធារណៈ សូមបង្កើត token ថ្មីតាម BotFather ហើយដាក់ token ថ្មីក្នុង `.env`។

## 5. អនុវត្ត Database migrations

Migration files ដែលបានបង្កើតរួចស្ថិតនៅក្នុង folder `drizzle/`។ បន្ទាប់ពីដាក់ `DATABASE_URL` ត្រឹមត្រូវ សូមរត់៖

```bash
pnpm drizzle-kit migrate
```

នេះនឹងអនុវត្ត SQL migration ទាំងអស់ទៅ database ដែលកំណត់ក្នុង `DATABASE_URL` និងបង្កើត tables ដូចជា `users`, `products`, និង `siteSettings`។

បើអ្នកបានកែ `drizzle/schema.ts` ថ្មីៗ ហើយចង់បង្កើត migration ថ្មី៖

```bash
pnpm drizzle-kit generate
pnpm drizzle-kit migrate
```

Project ក៏មាន shortcut៖

```bash
pnpm db:push
```

Script នេះរត់ `drizzle-kit generate` បន្ទាប់មក `drizzle-kit migrate`។ សម្រាប់ការដំឡើងដំបូងដែល migration files មានរួចហើយ `pnpm drizzle-kit migrate` គ្រប់គ្រាន់ និងសុវត្ថិភាពជាង។

បើ Drizzle បង្ហាញ error `DATABASE_URL is required` សូមពិនិត្យថា៖

1. `.env` នៅ root ដូចគ្នានឹង `package.json`។
2. ឈ្មោះ variable គឺ `DATABASE_URL` ត្រឹមត្រូវ។
3. មិនមាន space ឬ quote ខុសក្នុង connection string។
4. MySQL/TiDB កំពុងដំណើរការ និង user/password ត្រឹមត្រូវ។

## 6. Run application ក្នុង VS Code

Terminal 1៖

```bash
pnpm dev
```

បើជោគជ័យ នឹងឃើញ៖

```text
Server running on http://localhost:3000/
```

បើ port 3000 កំពុងប្រើ project នឹងរក port ទំនេរបន្ទាប់ដោយស្វ័យប្រវត្តិ។ បើចង់កំណត់ port ផ្សេង ប្តូរ `PORT` ក្នុង `.env`។

បើក browser ទៅ៖

```text
http://localhost:3000
```

Admin login និង feature ដែលត្រូវការ OAuth អាចត្រូវការកំណត់ redirect/callback URL សម្រាប់ local origin នៅក្នុង OAuth provider។ ប្រសិនបើ Manus OAuth មិនអនុញ្ញាត `http://localhost:3000` ក្នុង app របស់អ្នក អ្នកអាចរត់ storefront និង database local បាន ប៉ុន្តែត្រូវកំណត់ OAuth application/portal ឱ្យទទួល local callback មុននឹងប្រើ Admin login។

## 7. ពិនិត្យ build និង type errors

```bash
pnpm check
pnpm test
pnpm build
```

បើ `pnpm check` និង `pnpm build` ជោគជ័យ នោះ code និង dependency setup ត្រឹមត្រូវ។

## 8. Media និង uploads ពេលរត់ local

ផលិតផល/poster/video ចាស់ៗរបស់ deployment មាន media reference ដូចជា `/manus-storage/...` ដែលពឹងលើ Manus storage proxy។ Database local នឹងមានតែ record ប៉ុណ្ណោះ មិនមាន file bytes ទាំងអស់ដោយស្វ័យប្រវត្តិទេ។ ដូច្នេះ៖

- storefront និង database អាចរត់ local បាន ប្រសិនបើ Manus API/storage variables នៅត្រឹមត្រូវ;
- បើ media មិនបង្ហាញ សូមពិនិត្យ `BUILT_IN_FORGE_API_URL`, `BUILT_IN_FORGE_API_KEY` និង frontend forge variables;
- បើចង់ local 100% ដោយមិនពឹង Manus storage ត្រូវប្តូរ storage implementation ទៅ S3-compatible bucket ឬ local upload server ហើយ migrate media paths ដាច់ដោយឡែក។ កុំដាក់រូបភាព ឬវីដេអូធំៗក្នុង `client/public`។

## 9. សុវត្ថិភាព និង Git

បន្ថែមឯកសារទាំងនេះទៅ `.gitignore` មុន commit៖

```gitignore
.env
.env.*
!.env.example
```

កុំ commit៖ `DATABASE_URL`, `JWT_SECRET`, Telegram token, OAuth/API keys ឬ cloud credentials។ ប្រើ `.env` ក្នុងកុំព្យូទ័ររបស់អ្នកតែប៉ុណ្ណោះ។

## Quick start សង្ខេប

```bash
# នៅ root folder ដែលមាន package.json
pnpm install
# បង្កើត .env និងបំពេញ values
pnpm drizzle-kit migrate
pnpm check
pnpm dev
```

បន្ទាប់ពី app ដំណើរការ សូមបើក `http://localhost:3000` ហើយសាកល្បង storefront មុន បន្ទាប់មកសាកល្បង Admin login, product upload និង Telegram order notification។
