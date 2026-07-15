# Leak Hub: phone sync setup

One-time setup to make `leak-hub.html` sync between your laptop and phone.
About 20 minutes. Steps marked **[you]** only you can do (they need your logins).

The model in one line: the hub file lives on the web (Vercel), your typed content
lives in a private Supabase database, and row-level security means the database
will only ever hand your data to *you*, logged in with your email + password.

---

## 1. Create the Supabase project **[you]**

1. Go to https://supabase.com → sign in → **New project**.
2. Name it `leak-hub` (kept fully separate from your main Leak app).
3. Pick a region near you (Sydney), set a strong database password, create.
4. Wait ~2 min for it to spin up.

## 2. Create the storage table + security **[you]**

In the project: left sidebar → **SQL Editor** → **New query** → paste this, run it:

```sql
-- One row per user, holding the whole hub as JSON.
create table public.hub_state (
  user_id    uuid primary key references auth.users(id) on delete cascade,
  data       jsonb not null default '{}',
  updated_at timestamptz not null default now()
);

-- Lock the table down, then allow each user to touch ONLY their own row.
alter table public.hub_state enable row level security;

create policy "own row - read"   on public.hub_state
  for select using (auth.uid() = user_id);
create policy "own row - insert" on public.hub_state
  for insert with check (auth.uid() = user_id);
create policy "own row - update" on public.hub_state
  for update using (auth.uid() = user_id) with check (auth.uid() = user_id);
```

You should see “Success. No rows returned.”

## 3. Create your login **[you]**

Left sidebar → **Authentication** → **Users** → **Add user** → **Create new user**.
- Email: your email (e.g. hello@theserviceedit.com)
- Password: choose one you'll remember
- Tick **Auto Confirm User** (so no confirmation email is needed)

This is the account you'll sign into the hub with.

## 4. Grab your two keys **[you]**

Left sidebar → **Project Settings** → **API**. Copy:
- **Project URL** → looks like `https://abcdxyz.supabase.co`
- **anon public** key → a long string starting `eyJ...`

(The anon key is *designed* to be public. The security lives in the RLS policies
above, not in hiding the key.)

## 5. Paste the keys into the hub

Open `leak-hub.html`, find the CONFIG block near the top, and replace the two
placeholders with your values:

```js
window.LEAK_CONFIG = {
  SUPABASE_URL:  "https://abcdxyz.supabase.co",   // your Project URL
  SUPABASE_ANON: "eyJhbGciOi..."                  // your anon public key
};
```

Save the file. (Tell me the two values and I'll paste them in for you if you'd rather.)

## 6. Put it online (Vercel)

Two options:

**A. Simplest, served by your existing site.**
Move the file into the app's `public/` folder and push:

```
mv leak-hub.html public/leak-hub.html
git add public/leak-hub.html && git commit -m "Deploy Leak Hub with sync"
git push origin master
```

Vercel redeploys automatically. Your hub is then at:
`https://<your-leak-domain>/leak-hub.html`

**B. Its own address.** Tell me and I'll set it up as a standalone Vercel
project so it lives at something like `hub.theserviceedit.com`.

## 7. Add to your phone home screen

On your phone, open the hub URL in Safari (iPhone) or Chrome (Android):
- **iPhone:** Share button → **Add to Home Screen**
- **Android:** ⋮ menu → **Add to Home screen**

Sign in once with the email + password from step 3. It stays signed in and now
looks and opens like an app. Do the same on your laptop. Edit on either, and it
syncs through Supabase within a second.

---

## How it behaves

- **Offline / no signal:** keeps working, saves locally, syncs when you're back.
  The sidebar shows the status: *Synced*, *Saving…*, or *Offline (saved locally)*.
- **Two devices at once:** last save wins. As a solo user you'll rarely hit this,
  but don't edit the same field on both phones simultaneously and expect a merge.
- **Before you finish setup:** the hub still works fully as a local file on one
  device. It just won't sync. Nothing is lost when you switch sync on; your local
  content is pushed up the first time you sign in.
- **Backups:** the **Export backup** link still works and gives you a JSON copy
  anytime. Cheap insurance.
