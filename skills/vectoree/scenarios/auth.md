# App Auth (your users)

Use this playbook for **C09**. Parent skill: `SKILL.md` in this folder.

**Not the same as logging into Vectoree Cloud.** If the developer said "登录一下 Vectoree" / `vectoree login` / open the Cloud dashboard, that is **S01**. Stop this playbook.

Need a project API key first (`scenarios/connect.md`). Need `@vectoree/cli` ≥ 0.1.10 (`npx`).

---

## Hard rules (C09 first)

1. Identity lives in Vectoree `auth.users`. **Do not** `CREATE TABLE users` or `accounts`.
2. On business tables, `user_id` = Auth JWT `sub`.
3. Do not call `POST /api/auth/signup`. That is Cloud console registration.
4. Do not copy `/dashboard/login`.
5. Do not write OAuth Client Secret into the repo. Do not use Vectoree's platform Google/GitHub keys for the app.
6. Email verification is on by default (8-digit code). Mail for app users is Resend (org wallet $0.0005). Console mail is Feishu SMTP and is none of this playbook.

---

## C09: add login to *my* app

```bash
npx @vectoree/cli auth status
npx @vectoree/cli docs get auth-sdk
```

Email + password is already on. Google/GitHub: if `auth status` shows no oauth providers, **stop and tell the owner** to open:

- https://vectoree.ai/dashboard/authentication/auth-methods
- https://docs.vectoree.ai/auth

or run `npx @vectoree/cli auth open`. They paste **their** Client ID/Secret. Then continue.

```bash
npx @vectoree/cli auth snippet --ui
npx @vectoree/cli auth snippet --no-ui --lang ts
```

`--ui` is a login page the agent pastes into the app. `--no-ui` is REST for a backend. There is no hosted Auth UI.

Runtime paths (publishable `pk_...` until the user has a JWT):

- `POST /api/auth/users`
- `POST /api/auth/sessions`
- `POST /api/auth/email/send-verification`
- `POST /api/auth/email/verify` (8-digit `otp`)
- `GET /api/auth/oauth/google` or `/github` (only after the owner configured them)
- `GET /api/auth/methods`

C09 done when the app can register, verify the 8-digit code, and keep `user_id` = JWT `sub` on its own tables.
