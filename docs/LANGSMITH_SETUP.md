# LangSmith setup (md-LangChain fork)

This fork tracks [`langchain-ai/open-swe`](https://github.com/langchain-ai/open-swe) via `.github/workflows/sync-upstream.yml`.  
When LangSmith has **Automatically update on push** enabled, each sync rebuilds your deployments.

Robin needs **two** Agent Servers from this repo:

| Deployment | What it is | Project path |
| --- | --- | --- |
| **Backend** | Dashboard, webhooks, API | repo root (`langgraph.json`) |
| **MDA runtime** | Model loop + sandboxes | `mda/` (Managed Deep Agent) |

Full env and GitHub/Slack app details: [INSTALLATION.md](./INSTALLATION.md) and [mda/README.md](../mda/README.md).

---

## 0. Prerequisites (you)

- [ ] LangSmith workspace on a plan that supports Agent Servers
- [ ] Managed Deep Agents access (required for the `mda/` deployment)
- [ ] Org/admin access to authorize LangSmith’s GitHub app for `md-LangChain/open-swe`
- [ ] Ability to create a GitHub App and a Slack app for your team

---

## 1. Connect GitHub to LangSmith

1. LangSmith → **Agent Servers** → **+ Agent Server**
2. Authorize **hosted-langserve** if prompted (org owner/admin once)
3. Confirm the app can see `md-LangChain/open-swe` (GitHub → Settings → Applications → `hosted-langserve`)

---

## 2. Create the backend Agent Server

1. Source: **GitHub** → `md-LangChain/open-swe`
2. Branch: `main`
3. Config file: `langgraph.json`
4. Enable **Automatically update Agent Server on push to branch**
5. Prefer **Dedicated** if you need always-on workers (see INSTALLATION)
6. Submit, then copy the public URL → this becomes `LANGGRAPH_URL`

Initial revision may sit idle until secrets are set (step 4).

---

## 3. Create the MDA Agent Server

1. **+ Agent Server** again → same repo, branch `main`
2. Check **This is a Managed Deep Agent project**
3. Project directory: `mda`
4. Enable **Automatically update on push**
5. Add MDA secrets (minimum):
   - `OPEN_SWE_BACKEND_URL` = backend URL from step 2
   - `LANGSMITH_API_KEY` / gateway key as in [mda/README.md](../mda/README.md)
   - `MDA_SERVICE_SECRET` = shared secret you generate (`openssl rand -hex 32`)
6. Submit and copy the MDA URL → `MDA_RUNTIME_URL` for the backend

---

## 4. Finish backend secrets + apps

Follow [INSTALLATION.md](./INSTALLATION.md) steps 3–6:

1. Create the **GitHub App** (callback + webhook URLs use your deployment URL)
2. Create the **Slack app** from the manifest (same URL)
3. Set backend env vars, including at least:
   - `LANGGRAPH_URL`
   - `MDA_RUNTIME_URL`, `MDA_RUNTIME_API_KEY`, `REMOTE_RUNTIME_TOKEN_SECRET`, `MDA_SERVICE_SECRET`
   - GitHub App + Slack vars
   - `ALLOWED_GITHUB_ORGS` or `ALLOWED_GITHUB_USERS`
   - `CONFIGURED_ADMINS`
   - `TOKEN_ENCRYPTION_KEY`, `DASHBOARD_JWT_SECRET`
   - `LANGSMITH_GATEWAY_API_KEY` (key with `gateway:invoke`)

Saving env vars rolls a new revision.

---

## 5. Verify

- [ ] Open the backend URL → Sign in with GitHub
- [ ] Admin Defaults → set default repository
- [ ] Slack: `@Robin` in a channel
- [ ] GitHub: mention the configured handle on an issue

---

## Keeping up to date

```text
langchain-ai/open-swe  --(sync Action)-->  md-LangChain/open-swe@main
                                                    |
                                                    | LangSmith auto-update
                                                    v
                                          backend + MDA rebuild
```

- **Automatic:** sync workflow runs every 6 hours (and on **Actions → Sync upstream → Run workflow**). It resets to upstream `main`, then re-applies this checklist + the sync workflow so they are not deleted.
- **Manual:** prefer running the Action. A raw `git reset --hard upstream/main && git push` drops the fork overlay until the Action runs again.
- Keep **backend and MDA** on the same commit when possible
- Upstream may add required env vars; re-check INSTALLATION after large syncs
