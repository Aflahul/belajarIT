# Foundation, Auth, and Academics Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Menghasilkan aplikasi Next.js yang dapat dijalankan, memiliki autentikasi tiga peran, model akademik inti, isolasi akses kelas, dan panel administrasi dasar.

**Architecture:** Aplikasi memakai modular monolith Next.js App Router. Domain auth dan academics dipisah dalam modul terfokus; server actions hanya menjadi adapter tipis menuju service yang dapat diuji. PostgreSQL adalah sumber kebenaran, runtime memakai pooled Neon connection dan migrasi memakai direct connection.

**Tech Stack:** Node.js 24.x, npm 11.x, Next.js 16.x, React 19.x, TypeScript 5.x strict, PostgreSQL, Drizzle ORM, Zod 4.x, Vitest, Testing Library, Playwright, ESLint, Vercel.

**Spec:** `docs/product-spec.md`; detail teknis di `docs/architecture.md` dan UI di `docs/design-system.md`.

## Global Constraints

- Gunakan branch `dev` atau feature branch dari `dev`; jangan menulis ke `produksi` selama delivery ini.
- `package.json` wajib menetapkan `"engines": { "node": "24.x" }` dan versi aplikasi `0.1.0`.
- Gunakan Next.js App Router dengan Node.js runtime; jangan memakai deprecated Edge Functions.
- Runtime database menggunakan pooled `DATABASE_URL`; migrasi menggunakan direct `DATABASE_URL_UNPOOLED`.
- Google Apps Script dan Google Sheets lama tidak boleh dimodifikasi atau disinkronkan.
- Role hanya `ADMIN`, `GURU`, dan `MURID`.
- Login murid: kode kelas + nomor absen + PIN. Login guru/admin: email atau username + password.
- Semua otorisasi diperiksa pada server; proxy hanya prefilter navigasi.
- Password/PIN memakai scrypt, salt unik, dan pepper dari environment.
- Waktu disimpan dalam UTC dan ditampilkan sebagai `Asia/Jakarta`.
- Area murid mobile-first Comic Neobrutalism Edukatif; guru/admin Academic Calm.
- Tulis test gagal terlebih dahulu untuk setiap aturan bisnis, jalankan untuk membuktikan gagal, implementasikan minimal, lalu jalankan kembali.
- Setiap task berakhir dengan lint, type-check, test terkait, dan commit kecil.

## File Map

```text
src/
  app/
    (auth)/login/
    (student)/murid/
    (staff)/guru/
    (staff)/admin/
    api/health/
    version/
  components/
    auth/
    layout/
    ui/
  db/
    schema/
    client.ts
    index.ts
  lib/
    env.ts
    result.ts
    version.ts
  modules/
    auth/
    academics/
  styles/
    tokens.css
test/
  db/
  e2e/
  setup.ts
scripts/
  create-admin.ts
drizzle/
```

`src/modules/auth` memiliki credential, session, login, dan guard. `src/modules/academics` memiliki periode, kelas, enrollment, penugasan guru, policy, repository, dan service. Folder `app` tidak berisi aturan bisnis.

---

### Task 1: Application Foundation and Quality Gates

**Files:**
- Create: `package.json`
- Create: `package-lock.json` melalui `npm install`
- Create: `.nvmrc`
- Create: `.env.example`
- Create: `.gitignore`
- Create: `tsconfig.json`
- Create: `next-env.d.ts`
- Create: `next.config.ts`
- Create: `eslint.config.mjs`
- Create: `vitest.config.ts`
- Create: `test/setup.ts`
- Create: `src/app/layout.tsx`
- Create: `src/app/page.tsx`
- Create: `src/app/globals.css`
- Create: `src/styles/tokens.css`
- Create: `src/lib/version.test.ts`
- Create: `src/lib/version.ts`
- Create: `src/app/version/route.ts`

**Interfaces:**
- Produces: `getAppVersion(): { version: string; commit: string | null }`.
- Produces scripts: `dev`, `build`, `lint`, `typecheck`, `test`, `test:run`, `test:e2e`, `db:generate`, `db:migrate`.

- [ ] **Step 1: Create package manifest and install the locked toolchain**

Create `package.json`:

```json
{
  "name": "belajarit",
  "version": "0.1.0",
  "private": true,
  "engines": { "node": "24.x" },
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "eslint .",
    "typecheck": "tsc --noEmit",
    "test": "vitest",
    "test:run": "vitest run",
    "test:e2e": "playwright test",
    "db:generate": "drizzle-kit generate",
    "db:migrate": "drizzle-kit migrate"
  }
}
```

Run:

```bash
npm install next@16 react@19 react-dom@19 zod@4 drizzle-orm pg
npm install -D typescript@5 @types/node@24 @types/react @types/react-dom @types/pg eslint eslint-config-next vitest jsdom @testing-library/react @testing-library/jest-dom @playwright/test drizzle-kit tsx cross-env dotenv
```

Expected: `package-lock.json` exists and `npm ls --depth=0` exits 0.

- [ ] **Step 2: Add strict TypeScript, lint, Vitest, environment template, and design tokens**

Set `.nvmrc` to `24`. Set `tsconfig.json` to strict mode with alias `@/* -> ./src/*`. Set Vitest environment to `jsdom`, load `test/setup.ts`, exclude `test/e2e`, and provide test-only values for `AUTH_SECRET`, `AUTH_PEPPER`, `DATABASE_URL`, and `DATABASE_URL_UNPOOLED` through the Vitest `test.env` option. `.env.example` must contain names only:

```dotenv
DATABASE_URL=
DATABASE_URL_UNPOOLED=
AUTH_SECRET=
AUTH_PEPPER=
APP_VERSION=0.1.0
VERCEL_GIT_COMMIT_SHA=
```

Copy the approved `--neo-*` and `--calm-*` tokens from `docs/design-system.md` into `src/styles/tokens.css`. Import tokens from `src/app/globals.css`.

- [ ] **Step 3: Write the failing version metadata test**

Create `src/lib/version.test.ts`:

```ts
import { describe, expect, it, vi } from "vitest";
import { getAppVersion } from "./version";

describe("getAppVersion", () => {
  it("returns package version and the short deployment commit", () => {
    vi.stubEnv("APP_VERSION", "0.1.0");
    vi.stubEnv("VERCEL_GIT_COMMIT_SHA", "89f67b752b79a67f");
    expect(getAppVersion()).toEqual({ version: "0.1.0", commit: "89f67b7" });
    vi.unstubAllEnvs();
  });
});
```

- [ ] **Step 4: Run the version test to verify it fails**

Run: `npm run test:run -- src/lib/version.test.ts`

Expected: FAIL because `src/lib/version.ts` does not exist.

- [ ] **Step 5: Implement version metadata and endpoint**

Create `src/lib/version.ts`:

```ts
export function getAppVersion() {
  const version = process.env.APP_VERSION ?? "0.1.0";
  const sha = process.env.VERCEL_GIT_COMMIT_SHA;
  return { version, commit: sha ? sha.slice(0, 7) : null };
}
```

Create `src/app/version/route.ts`:

```ts
import { getAppVersion } from "@/lib/version";

export const runtime = "nodejs";

export async function GET() {
  return Response.json(getAppVersion(), {
    headers: { "Cache-Control": "no-store" },
  });
}
```

Create a minimal root layout and page that state the app is under development and display the version in the footer.

- [ ] **Step 6: Verify foundation quality gates**

Run:

```bash
npm run test:run -- src/lib/version.test.ts
npm run lint
npm run typecheck
npm run build
```

Expected: all commands exit 0; version test reports 1 passed; build lists `/` and `/version`.

- [ ] **Step 7: Commit foundation**

```bash
git add package.json package-lock.json .nvmrc .env.example .gitignore tsconfig.json next-env.d.ts next.config.ts eslint.config.mjs vitest.config.ts test src
git commit -m "chore: scaffold Next.js application foundation"
```

---

### Task 2: Database Connection and Academic Schema

**Files:**
- Create: `drizzle.config.ts`
- Create: `compose.test.yml`
- Create: `src/lib/env.ts`
- Create: `src/lib/env.test.ts`
- Create: `src/db/client.ts`
- Create: `src/db/index.ts`
- Create: `src/db/schema/enums.ts`
- Create: `src/db/schema/users.ts`
- Create: `src/db/schema/academics.ts`
- Create: `src/db/schema/index.ts`
- Create: generated files under `drizzle/`
- Create: `test/db/client.ts`
- Create: `test/db/academic-constraints.test.ts`

**Interfaces:**
- Produces: `env.DATABASE_URL`, `env.DATABASE_URL_UNPOOLED`, `env.AUTH_SECRET`, `env.AUTH_PEPPER`.
- Produces: `db` Drizzle client using the module-scoped `pg.Pool`.
- Produces tables: `users`, `students`, `academicPeriods`, `classes`, `classEnrollments`, `teacherClassAssignments`.

- [ ] **Step 1: Write the failing environment validation test**

```ts
import { describe, expect, it } from "vitest";
import { parseServerEnv } from "./env";

describe("parseServerEnv", () => {
  it("rejects missing auth secrets", () => {
    expect(() => parseServerEnv({ DATABASE_URL: "postgres://example" })).toThrow(
      "AUTH_SECRET",
    );
  });
});
```

- [ ] **Step 2: Run the environment test to verify it fails**

Run: `npm run test:run -- src/lib/env.test.ts`

Expected: FAIL because `parseServerEnv` is undefined.

- [ ] **Step 3: Implement server environment parsing**

```ts
import { z } from "zod";

const serverEnvSchema = z.object({
  DATABASE_URL: z.string().url(),
  DATABASE_URL_UNPOOLED: z.string().url(),
  AUTH_SECRET: z.string().min(32),
  AUTH_PEPPER: z.string().min(32),
});

export function parseServerEnv(input: NodeJS.ProcessEnv) {
  return serverEnvSchema.parse(input);
}

export const env = parseServerEnv(process.env);
```

Do not import `env.ts` from Client Components.

- [ ] **Step 4: Define the six-table Drizzle schema**

Use UUID primary keys, timestamps with time zone, and these ownership rules:

```ts
export const userRole = pgEnum("user_role", ["ADMIN", "GURU", "MURID"]);
export const periodStatus = pgEnum("period_status", ["DRAFT", "ACTIVE", "ARCHIVED"]);
```

`users` columns: `id`, `role`, `displayName`, nullable `emailNormalized`, nullable `usernameNormalized`, nullable `passwordHash`, `credentialVersion` default 1, `failedLoginCount` default 0, nullable `lockedUntil`, `active` default true, `createdAt`, `updatedAt`. Add unique indexes for non-null normalized email and username.

`students` columns: `userId` primary key and FK to users, `studentCode` unique, `pinHash`, `createdAt`, `updatedAt`.

`academicPeriods` columns: `id`, `academicYear`, `semester`, `status`, nullable `startsAt`, nullable `endsAt`, `createdAt`, `updatedAt`. Add unique `(academicYear, semester)` and a partial unique index on `status` where status is `ACTIVE`.

`classes` columns: `id`, `academicPeriodId`, `code`, `name`, `active`, `createdAt`, `updatedAt`. Add unique `(academicPeriodId, code)`.

`classEnrollments` columns: `id`, `classId`, `studentUserId`, `attendanceNumber`, `active`, `createdAt`, `updatedAt`. Add unique `(classId, studentUserId)` and `(classId, attendanceNumber)`.

`teacherClassAssignments` columns: `id`, `classId`, `teacherUserId`, `active`, `createdAt`, `updatedAt`. Add unique `(classId, teacherUserId)`.

Every FK uses `onDelete: "restrict"` except one-to-one `students.userId`, which uses `cascade` only for unpersisted/test accounts; production UI never hard-deletes users.

- [ ] **Step 5: Implement pooled runtime client and direct migration config**

`src/db/client.ts`:

```ts
import { drizzle } from "drizzle-orm/node-postgres";
import { Pool } from "pg";
import { env } from "@/lib/env";
import * as schema from "./schema";

const globalForDb = globalThis as unknown as { belajarItPool?: Pool };
const pool = globalForDb.belajarItPool ?? new Pool({
  connectionString: env.DATABASE_URL,
  max: 5,
});

if (process.env.NODE_ENV !== "production") globalForDb.belajarItPool = pool;

export const db = drizzle({ client: pool, schema });
```

Create `drizzle.config.ts`:

```ts
import "dotenv/config";
import { defineConfig } from "drizzle-kit";

if (!process.env.DATABASE_URL_UNPOOLED) {
  throw new Error("DATABASE_URL_UNPOOLED is required for migrations");
}

export default defineConfig({
  schema: "./src/db/schema/index.ts",
  out: "./drizzle",
  dialect: "postgresql",
  dbCredentials: { url: process.env.DATABASE_URL_UNPOOLED },
  strict: true,
});
```

Create `compose.test.yml`:

```yaml
services:
  postgres:
    image: postgres:17-alpine
    environment:
      POSTGRES_DB: belajarit_test
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    ports:
      - "54329:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres -d belajarit_test"]
      interval: 2s
      timeout: 2s
      retries: 15
```

- [ ] **Step 6: Write constraint tests and verify the unmigrated database fails**

Start the empty test database:

Run:

```bash
docker compose -f compose.test.yml up -d
```

Create `test/db/client.ts` with a dedicated `pg.Pool` using `TEST_DATABASE_URL`, the same Drizzle schema, and an `afterAll` hook that closes the pool. Create `test/db/academic-constraints.test.ts` that truncates the six tables in FK-safe order before each test, inserts one period with `ACTIVE`, then expects insertion of a second `ACTIVE` period to reject. Add a second test that creates two enrollments with the same class and attendance number and expects the second insert to reject.

Run: `npx cross-env TEST_DATABASE_URL=postgres://postgres:postgres@localhost:54329/belajarit_test npm run test:run -- test/db/academic-constraints.test.ts`

Expected: FAIL because the empty test database does not contain `academic_periods`.

- [ ] **Step 7: Generate migration, migrate, and verify constraints pass**

Run:

```bash
npm run db:generate
npx cross-env DATABASE_URL_UNPOOLED=postgres://postgres:postgres@localhost:54329/belajarit_test npm run db:migrate
npx cross-env TEST_DATABASE_URL=postgres://postgres:postgres@localhost:54329/belajarit_test npm run test:run -- test/db/academic-constraints.test.ts
```

Expected: 2 tests pass. Then run `npm run lint && npm run typecheck` and expect exit 0.

- [ ] **Step 8: Commit schema and migrations**

```bash
git add drizzle.config.ts compose.test.yml src/lib/env.ts src/lib/env.test.ts src/db drizzle test/db package.json package-lock.json
git commit -m "feat: add academic database foundation"
```

---

### Task 3: Academic Policies and Services

**Files:**
- Create: `src/lib/result.ts`
- Create: `src/modules/academics/types.ts`
- Create: `src/modules/academics/schemas.ts`
- Create: `src/modules/academics/repository.ts`
- Create: `src/modules/academics/service.ts`
- Create: `src/modules/academics/service.test.ts`
- Create: `src/modules/academics/policy.ts`
- Create: `src/modules/academics/policy.test.ts`

**Interfaces:**
- Produces: `Result<T>` discriminated union.
- Produces: `createPeriod`, `activatePeriod`, `createClass`, `enrollStudent`, `assignTeacher`.
- Produces: `canAccessClass(actor, classId, repository): Promise<boolean>`.

- [ ] **Step 1: Define result and command contracts**

```ts
export type AppError = {
  code: string;
  message: string;
  fieldErrors?: Record<string, string[]>;
};

export type Result<T> =
  | { ok: true; data: T }
  | { ok: false; error: AppError };
```

Commands use Zod schemas with trimmed names, uppercase class codes, semester `1 | 2`, positive attendance number, and UUID identifiers.

- [ ] **Step 2: Write failing academic service tests**

Use an in-memory fake implementing `AcademicRepository`. Cover:

```ts
it("archives the previous active period when a new period is activated", async () => {
  const result = await service.activatePeriod(newPeriodId);
  expect(result.ok).toBe(true);
  expect(repo.periodStatus(oldPeriodId)).toBe("ARCHIVED");
  expect(repo.periodStatus(newPeriodId)).toBe("ACTIVE");
});

it("rejects a duplicate attendance number inside the same class", async () => {
  const result = await service.enrollStudent({ classId, studentUserId, attendanceNumber: 7 });
  expect(result).toMatchObject({ ok: false, error: { code: "ATTENDANCE_TAKEN" } });
});
```

- [ ] **Step 3: Run academic service tests to verify they fail**

Run: `npm run test:run -- src/modules/academics/service.test.ts`

Expected: FAIL because service functions are not implemented.

- [ ] **Step 4: Implement repository interface and transactional services**

Repository contract:

```ts
export interface AcademicRepository {
  createPeriod(input: CreatePeriodInput): Promise<AcademicPeriod>;
  activatePeriod(periodId: string): Promise<AcademicPeriod>;
  createClass(input: CreateClassInput): Promise<ClassRecord>;
  enrollStudent(input: EnrollStudentInput): Promise<ClassEnrollment>;
  assignTeacher(input: AssignTeacherInput): Promise<TeacherClassAssignment>;
  userHasClass(userId: string, role: UserRole, classId: string): Promise<boolean>;
}
```

The Drizzle implementation of `activatePeriod` must execute archive-old and activate-new in one database transaction. Translate known unique-constraint violations into `PERIOD_EXISTS`, `CLASS_CODE_EXISTS`, `STUDENT_ALREADY_ENROLLED`, or `ATTENDANCE_TAKEN`; do not expose raw SQL errors.

- [ ] **Step 5: Write failing class access policy tests**

```ts
it.each([
  ["ADMIN", true],
  ["GURU", true],
  ["MURID", true],
])("allows %s when repository confirms membership", async (role, expected) => {
  expect(await canAccessClass({ userId, role }, classId, repo)).toBe(expected);
});

it("denies a teacher without an active assignment", async () => {
  repo.userHasClassResult = false;
  expect(await canAccessClass({ userId, role: "GURU" }, classId, repo)).toBe(false);
});
```

- [ ] **Step 6: Implement policy and verify service suite**

`ADMIN` returns true for any existing class. `GURU` requires active `teacher_class_assignments`. `MURID` requires active `class_enrollments`. Run:

```bash
npm run test:run -- src/modules/academics
npm run lint
npm run typecheck
```

Expected: all academic tests pass and quality commands exit 0.

- [ ] **Step 7: Commit academic domain**

```bash
git add src/lib/result.ts src/modules/academics
git commit -m "feat: add academic policies and services"
```

---

### Task 4: Credential Hashing and Signed Sessions

**Files:**
- Create: `src/modules/auth/types.ts`
- Create: `src/modules/auth/secret.ts`
- Create: `src/modules/auth/secret.test.ts`
- Create: `src/modules/auth/session.ts`
- Create: `src/modules/auth/session.test.ts`
- Create: `src/modules/auth/cookies.ts`

**Interfaces:**
- Produces: `hashSecret(secret): Promise<string>`.
- Produces: `verifySecret(secret, encoded): Promise<boolean>`.
- Produces: `createSessionToken(payload, secret = env.AUTH_SECRET): string` and `verifySessionToken(token, secret = env.AUTH_SECRET, nowSeconds = currentUnixTime): SessionPayload | null`.
- Produces: `setSessionCookie`, `clearSessionCookie`, `readSessionCookie`.

- [ ] **Step 1: Write failing secret hashing tests**

```ts
it("hashes without retaining the raw PIN", async () => {
  const encoded = await hashSecret("1234");
  expect(encoded).toMatch(/^scrypt\$/);
  expect(encoded).not.toContain("1234");
  await expect(verifySecret("1234", encoded)).resolves.toBe(true);
  await expect(verifySecret("9999", encoded)).resolves.toBe(false);
});
```

- [ ] **Step 2: Run hashing tests to verify they fail**

Run: `npm run test:run -- src/modules/auth/secret.test.ts`

Expected: FAIL because hashing functions do not exist.

- [ ] **Step 3: Implement scrypt credential format**

Use `node:crypto` `randomBytes`, promisified `scrypt`, and `timingSafeEqual`. Store:

```text
scrypt$16384$8$1$<base64url-salt>$<base64url-derived-key>
```

Append `AUTH_PEPPER` to the supplied secret before deriving a 64-byte key. Reject malformed encoded values by returning `false`, not by throwing credential details.

- [ ] **Step 4: Write failing signed-session tests**

```ts
const payload = {
  userId: "9e7454a7-3411-4d7f-91b3-b2f391507058",
  role: "MURID" as const,
  credentialVersion: 1,
  expiresAt: 1_800_000_000,
};

it("round-trips a signed session and rejects tampering", () => {
  const token = createSessionToken(payload, "a".repeat(32));
  expect(verifySessionToken(token, "a".repeat(32), 1_700_000_000)).toEqual(payload);
  expect(verifySessionToken(`${token}x`, "a".repeat(32), 1_700_000_000)).toBeNull();
});
```

- [ ] **Step 5: Implement HMAC session tokens and cookies**

Token format is `<base64url-json>.<base64url-hmac-sha256>`. Validate payload with Zod after signature verification. Cookie name is `belajarit_session`; attributes are `httpOnly`, `secure` in production, `sameSite: "lax"`, `path: "/"`, and `maxAge: 43200` seconds.

- [ ] **Step 6: Verify auth primitive tests and commit**

Run:

```bash
npm run test:run -- src/modules/auth/secret.test.ts src/modules/auth/session.test.ts
npm run lint
npm run typecheck
```

Expected: hashing and session suites pass.

```bash
git add src/modules/auth
git commit -m "feat: add credential and session security"
```

---

### Task 5: Staff and Student Login Services

**Files:**
- Create: `src/modules/auth/schemas.ts`
- Create: `src/modules/auth/repository.ts`
- Create: `src/modules/auth/login-service.ts`
- Create: `src/modules/auth/login-service.test.ts`
- Create: `src/modules/auth/actions.ts`
- Create: `scripts/create-admin.ts`

**Interfaces:**
- Produces: `loginStaff(input, context: LoginContext): Promise<Result<AuthSession>>`.
- Produces: `loginStudent(input, context: LoginContext): Promise<Result<AuthSession>>`.
- Produces server actions: `loginStaffAction`, `loginStudentAction`, `logoutAction`.

- [ ] **Step 1: Write failing staff login tests**

Cover successful normalized email login, username login, wrong password, inactive account, and lockout:

```ts
it("locks an account for 15 minutes after the fifth failed attempt", async () => {
  for (let attempt = 1; attempt <= 5; attempt += 1) {
    await service.loginStaff(
      { identifier: "guru", password: "wrong" },
      { now },
    );
  }
  expect(repo.user.lockedUntil).toEqual(new Date(now.getTime() + 15 * 60_000));
});
```

Every failed public result must be `{ ok: false, error: { code: "INVALID_CREDENTIALS", message: "Data login tidak sesuai." } }` except malformed form fields, which return `VALIDATION_ERROR`.

- [ ] **Step 2: Write failing student login tests**

Cover active-period class code + attendance number + PIN, wrong PIN, inactive enrollment, and same attendance number in a different class. Confirm lookup never authenticates against an archived period.

- [ ] **Step 3: Run login tests to verify they fail**

Run: `npm run test:run -- src/modules/auth/login-service.test.ts`

Expected: FAIL because login services are not implemented.

- [ ] **Step 4: Implement auth repository and lockout behavior**

Repository contract:

```ts
export interface AuthRepository {
  findStaffByIdentifier(normalized: string): Promise<AuthUser | null>;
  findStudentByActiveClass(input: {
    classCode: string;
    attendanceNumber: number;
  }): Promise<AuthStudent | null>;
  recordFailedLogin(userId: string, now: Date): Promise<void>;
  clearFailedLogins(userId: string): Promise<void>;
  findUserSessionState(userId: string): Promise<{
    active: boolean;
    credentialVersion: number;
    role: UserRole;
  } | null>;
}
```

Normalize identifiers with `trim().toLowerCase()` and class codes with `trim().toUpperCase()`. At attempt 5, set `lockedUntil = now + 15 minutes`. A successful login resets the counter. Never log raw credentials.

- [ ] **Step 5: Implement login actions and admin bootstrap script**

Actions validate `FormData` with Zod, call the service, set a 12-hour session, and redirect by role only after success. `scripts/create-admin.ts` requires `ADMIN_EMAIL`, `ADMIN_USERNAME`, `ADMIN_NAME`, and `ADMIN_PASSWORD`, hashes the password, performs an upsert by normalized email, and prints only the created user ID.

Add script:

```json
"admin:create": "tsx scripts/create-admin.ts"
```

- [ ] **Step 6: Verify login suite and bootstrap against test database**

Run:

```bash
npm run test:run -- src/modules/auth
ADMIN_EMAIL=admin@example.test ADMIN_USERNAME=admin ADMIN_NAME=Admin ADMIN_PASSWORD='test-only-strong-password' npm run admin:create
```

Expected: auth tests pass; script prints one UUID and does not print the password.

- [ ] **Step 7: Commit login services**

```bash
git add src/modules/auth scripts/create-admin.ts package.json package-lock.json
git commit -m "feat: add staff and student login services"
```

---

### Task 6: Login UI, Server Guards, and Role Layouts

**Files:**
- Create: `src/proxy.ts`
- Create: `src/modules/auth/current-user.ts`
- Create: `src/modules/auth/guards.ts`
- Create: `src/modules/auth/guards.test.ts`
- Create: `src/app/(auth)/login/page.tsx`
- Create: `src/components/auth/staff-login-form.tsx`
- Create: `src/components/auth/student-login-form.tsx`
- Create: `src/components/auth/login-form.module.css`
- Create: `src/app/(student)/murid/layout.tsx`
- Create: `src/app/(student)/murid/page.tsx`
- Create: `src/app/(staff)/guru/layout.tsx`
- Create: `src/app/(staff)/guru/page.tsx`
- Create: `src/app/(staff)/admin/layout.tsx`
- Create: `src/app/(staff)/admin/page.tsx`
- Create: `src/components/layout/student-shell.tsx`
- Create: `src/components/layout/staff-shell.tsx`

**Interfaces:**
- Produces: `getCurrentUser(): Promise<CurrentUser | null>`.
- Produces: `authorizeRole(user, allowedRoles): boolean`, `requireUser()`, and `requireRole(allowedRoles)`.

- [ ] **Step 1: Write failing role guard tests**

```ts
it("rejects a student from an admin-only policy", () => {
  expect(authorizeRole(studentUser, ["ADMIN"])).toBe(false);
});

it("rejects a signed token after credentialVersion changes", async () => {
  repo.sessionState.credentialVersion = 2;
  await expect(getCurrentUserFromToken(versionOneToken, repo)).resolves.toBeNull();
});
```

- [ ] **Step 2: Run guard tests to verify they fail**

Run: `npm run test:run -- src/modules/auth/guards.test.ts`

Expected: FAIL because guard functions do not exist.

- [ ] **Step 3: Implement current-user verification and guards**

Verification order: read cookie → verify HMAC/expiry → fetch current account state → compare active, role, credentialVersion → return current user. `authorizeRole` is a pure boolean policy. `requireRole` redirects unauthenticated users to `/login` and wrong roles to their own dashboard.

`src/proxy.ts` may redirect obvious missing-cookie requests, but every protected layout must still call `requireRole` server-side.

Add static security headers through `next.config.ts`: `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, `Referrer-Policy: strict-origin-when-cross-origin`, and a restrictive `Permissions-Policy`. A nonce-based Content Security Policy is completed during Delivery 5 hardening because a static policy would break Next.js runtime scripts.

- [ ] **Step 4: Implement dual login UI and accessible error states**

Login page uses two tabs: `Murid` and `Guru/Admin`. Student fields are class, attendance number, and PIN. Staff fields are email/username and password. Use approved design tokens, visible labels, 44px targets, `aria-describedby` for errors, pending button text, and generic invalid-credential copy.

- [ ] **Step 5: Implement protected role layouts and empty dashboards**

Student shell uses Comic Neobrutalism with item navigasi awal `Beranda`, `Belajar`, `Tugas`, `Riwayat`. Staff shell uses Academic Calm sidebar. Each page displays current name, role-appropriate empty state, version footer, and logout.

- [ ] **Step 6: Test rendered login forms and role guards**

Add Testing Library assertions that every field has an accessible label and that the pending state disables double submit. Run:

```bash
npm run test:run -- src/modules/auth src/components/auth
npm run lint
npm run typecheck
npm run build
```

Expected: tests pass; build exposes `/login`, `/murid`, `/guru`, and `/admin`.

- [ ] **Step 7: Commit authentication UI**

```bash
git add src/proxy.ts src/modules/auth src/app src/components src/styles
git commit -m "feat: add role-based login and protected layouts"
```

---

### Task 7: Admin Academic Management

**Files:**
- Create: `src/modules/academics/actions.ts`
- Create: `src/modules/academics/queries.ts`
- Create: `src/modules/academics/account-service.ts`
- Create: `src/modules/academics/account-service.test.ts`
- Create: `src/app/(staff)/admin/periode/page.tsx`
- Create: `src/app/(staff)/admin/kelas/page.tsx`
- Create: `src/app/(staff)/admin/pengguna/page.tsx`
- Create: `src/components/academics/period-form.tsx`
- Create: `src/components/academics/class-form.tsx`
- Create: `src/components/academics/user-form.tsx`
- Create: `src/components/academics/enrollment-form.tsx`
- Create: `src/components/academics/teacher-assignment-form.tsx`
- Create: `src/components/ui/form-message.tsx`
- Create: `src/components/ui/data-table.tsx`

**Interfaces:**
- Produces admin-only actions: `createPeriodAction`, `activatePeriodAction`, `createClassAction`, `createStudentAction`, `createTeacherAction`, `enrollStudentAction`, `assignTeacherAction`, `resetStudentPinAction`.
- Produces read models for period, class, user, enrollment, and assignment lists.

- [ ] **Step 1: Write failing account creation and PIN reset tests**

```ts
it("creates a student user and profile in one transaction", async () => {
  const result = await service.createStudent({
    displayName: "Murid Uji",
    studentCode: "XRPL1-99",
    pin: "1234",
  });
  expect(result.ok).toBe(true);
  expect(repo.transactionCount).toBe(1);
  expect(repo.student.pinHash).toMatch(/^scrypt\$/);
});

it("increments credentialVersion when an admin resets a PIN", async () => {
  await service.resetStudentPin(studentUserId, "5678");
  expect(repo.user.credentialVersion).toBe(2);
});
```

- [ ] **Step 2: Run account service tests to verify they fail**

Run: `npm run test:run -- src/modules/academics/account-service.test.ts`

Expected: FAIL because account service does not exist.

- [ ] **Step 3: Implement transactional account services and admin actions**

Create user + student profile atomically. Create teacher with role `GURU` and password hash. PIN reset updates `students.pinHash`, increments `users.credentialVersion`, clears lockout fields, and writes no plaintext value. Every action starts with `requireRole(["ADMIN"])` and returns `Result` for expected validation/domain errors.

- [ ] **Step 4: Implement period and class management pages**

`/admin/periode` lists DRAFT/ACTIVE/ARCHIVED and requires confirmation before activation. `/admin/kelas` filters by period, creates classes, shows enrollment counts, and links enrollment/assignment controls. Empty states include a single primary action.

- [ ] **Step 5: Implement user, enrollment, and assignment pages**

`/admin/pengguna` separates murid and staf, never displays credential hashes, and supports create, deactivate, PIN reset, enroll, and teacher assignment. Do not implement CSV/Sheets import in this delivery.

- [ ] **Step 6: Verify admin behavior**

Add component tests for validation messages and action tests confirming a non-admin receives access denied. Run:

```bash
npm run test:run -- src/modules/academics src/components/academics
npm run lint
npm run typecheck
npm run build
```

Expected: tests and quality gates pass; admin routes build successfully.

- [ ] **Step 7: Commit admin academics**

```bash
git add src/modules/academics 'src/app/(staff)/admin' src/components/academics src/components/ui
git commit -m "feat: add academic administration"
```

---

### Task 8: End-to-End Isolation, CI, and Delivery Handoff

**Files:**
- Create: `playwright.config.ts`
- Create: `test/e2e/auth.setup.ts`
- Create: `test/e2e/role-isolation.spec.ts`
- Create: `test/e2e/admin-academics.spec.ts`
- Create: `scripts/seed-e2e.ts`
- Create: `.github/workflows/ci.yml`
- Create: `src/app/api/health/route.ts`
- Modify: `README.md`
- Modify: `docs/implementation-roadmap.md`

**Interfaces:**
- Produces: `/api/health` returning `{ status: "ok", database: "reachable", version }` without secret details.
- Produces reproducible E2E accounts only in the test database.

- [ ] **Step 1: Write failing role-isolation E2E tests**

Test these browser behaviors:

```ts
test("student cannot open admin routes", async ({ page }) => {
  await loginAsStudent(page);
  await page.goto("/admin");
  await expect(page).toHaveURL(/\/murid$/);
});

test("teacher cannot open an unassigned class", async ({ page }) => {
  await loginAsTeacher(page);
  await page.goto(`/guru/kelas/${unassignedClassId}`);
  await expect(page.getByText("Anda tidak memiliki akses ke kelas ini.")).toBeVisible();
});
```

Also test student lookup by correct class + attendance number, generic wrong-PIN error, admin period activation, and logout.

- [ ] **Step 2: Run E2E tests to verify at least one fails**

Run:

```bash
npm run dev
npm run test:e2e -- test/e2e/role-isolation.spec.ts
```

Expected before final wiring: at least the unassigned-class scenario fails because the class detail denial page is missing.

- [ ] **Step 3: Implement missing class denial route and health check**

Create the minimal `/guru/kelas/[classId]` page that calls `canAccessClass`; render an explicit denial state instead of leaking class data. Health route executes `select 1`, returns HTTP 200 only when reachable, and never returns the connection string or raw database error.

- [ ] **Step 4: Add deterministic E2E seed and CI**

`scripts/seed-e2e.ts` recreates only records marked with `e2e-` codes in the test database: one active period, two classes, admin, assigned teacher, unassigned class, and two students. Add script `test:seed:e2e`.

Create `.github/workflows/ci.yml`:

```yaml
name: CI

on:
  pull_request:
  push:
    branches: [dev]

env:
  DATABASE_URL: postgres://postgres:postgres@localhost:5432/belajarit_test
  DATABASE_URL_UNPOOLED: postgres://postgres:postgres@localhost:5432/belajarit_test
  TEST_DATABASE_URL: postgres://postgres:postgres@localhost:5432/belajarit_test
  AUTH_SECRET: ci-auth-secret-at-least-32-characters
  AUTH_PEPPER: ci-auth-pepper-at-least-32-characters
  APP_VERSION: 0.1.0

jobs:
  quality:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:17-alpine
        env:
          POSTGRES_DB: belajarit_test
          POSTGRES_USER: postgres
          POSTGRES_PASSWORD: postgres
        ports: ["5432:5432"]
        options: >-
          --health-cmd "pg_isready -U postgres -d belajarit_test"
          --health-interval 5s --health-timeout 5s --health-retries 10
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 24
          cache: npm
      - run: npm ci
      - run: npm run db:migrate
      - run: npm run lint
      - run: npm run typecheck
      - run: npm run test:run
      - run: npm run build

  e2e:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:17-alpine
        env:
          POSTGRES_DB: belajarit_test
          POSTGRES_USER: postgres
          POSTGRES_PASSWORD: postgres
        ports: ["5432:5432"]
        options: >-
          --health-cmd "pg_isready -U postgres -d belajarit_test"
          --health-interval 5s --health-timeout 5s --health-retries 10
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 24
          cache: npm
      - run: npm ci
      - run: npm run db:migrate
      - run: npm run test:seed:e2e
      - run: npx playwright install --with-deps chromium
      - run: npm run test:e2e
```

Create `playwright.config.ts` with `testDir: "./test/e2e"`, Chromium as the only phase-one browser, `baseURL: "http://127.0.0.1:3000"`, trace on first retry, screenshot on failure, and this web server:

```ts
webServer: {
  command: "npm run dev",
  url: "http://127.0.0.1:3000",
  reuseExistingServer: !process.env.CI,
  timeout: 120_000,
}
```

- [ ] **Step 5: Run the full local verification**

Run:

```bash
npm run lint
npm run typecheck
npm run test:run
npm run build
npm run test:e2e
```

Expected: every command exits 0; Vitest and Playwright report zero failures.

- [ ] **Step 6: Update delivery documentation**

README must document local setup, environment variable names, migration commands, admin bootstrap, and test commands. Mark Delivery 1 completed only after the verification output above. Record that Delivery 2 is the next plan; do not claim learning content exists yet.

- [ ] **Step 7: Commit delivery verification and CI**

```bash
git add .github playwright.config.ts test/e2e scripts/seed-e2e.ts src/app/api 'src/app/(staff)/guru' README.md docs/implementation-roadmap.md package.json package-lock.json
git commit -m "test: verify foundation role isolation"
```

## Delivery Acceptance Checklist

- [ ] Admin, guru, dan murid dapat login dengan identifier yang disetujui.
- [ ] Percobaan kelima mengunci akun selama 15 menit; pesan publik tetap generik.
- [ ] Reset credential membatalkan token lama melalui `credentialVersion`.
- [ ] Satu periode aktif dan nomor absen unik per kelas dipaksa database.
- [ ] Guru hanya dapat mengakses kelas dengan assignment aktif.
- [ ] Murid hanya dapat mengakses kelas dengan enrollment aktif.
- [ ] Admin dapat mengelola periode, kelas, akun, enrollment, dan assignment.
- [ ] Student shell dan staff shell memakai sistem visual yang berbeda sesuai design system.
- [ ] `/version` dan `/api/health` tersedia tanpa membocorkan secret.
- [ ] Lint, type-check, unit, integration, database, build, dan E2E test lulus.
- [ ] Apps Script dan spreadsheet lama tidak berubah.

## Verified Technical References

- Next.js installation and Node.js requirements: https://nextjs.org/docs/app/getting-started/installation
- Vercel supported Node.js versions: https://vercel.com/docs/functions/runtimes/node-js/node-js-versions
- Neon connection guidance for Drizzle: https://neon.com/docs/guides/drizzle
- Drizzle PostgreSQL connection drivers: https://orm.drizzle.team/docs/get-started-postgresql
