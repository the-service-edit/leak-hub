# Leak Hub

Personal command centre for the Leak business. A single-page app: your thinking
(Goals & Concepts, and more pages as they're built) synced across laptop and phone.

- **Front end:** one static `index.html`, no build step.
- **Sync + auth:** Supabase (separate project), row-level security so data is
  private to the signed-in user.
- **Hosting:** Vercel (static, serves `index.html` at the root).

## Turn on sync
See **LEAK-HUB-SETUP.md**: create the Supabase project, run the SQL, add your
user, paste the two keys into the CONFIG block near the top of `index.html`.

Until configured it runs in local-only mode (saves to the browser on one device).
