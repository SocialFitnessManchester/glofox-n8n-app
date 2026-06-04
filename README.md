# n8n-nodes-glofox

Custom n8n community node for Glofox (Social Fitness). Port of the existing private Zapier app to n8n, with one major upgrade: the **Studio dropdown is sheet-backed**, so adding a new gym = adding a row in the studio-config sheet, not creating yet another n8n credential.

## What it does

A single **Glofox** node exposing two resources, each with a `Create` operation:

| Resource | Operation | What it does |
|---|---|---|
| **Lead** | Create | Creates a new lead (contact) in the selected studio's Glofox branch. Calls `POST /2.1/branches/{branchId}/leads`. |
| **Purchase** | Create | Looks up an existing contact by email, then assigns a membership plan. Calls `GET /2.0/members?email=...` then `POST /2.2/branches/{branchId}/users/{userId}/memberships/{membershipId}/plans/{planCode}/purchase`. |

Three dynamic dropdowns make it human-proof:

1. **Studio** — populated from your studio config Google Sheet (the same one used by the Glofox → GHL automation in [SocialFitnessManchester/automations](https://github.com/SocialFitnessManchester/automations))
2. **Membership Name** (Purchase only) — populated live from Glofox via `GET /2.0/memberships` for the selected studio
3. **Plan Name** (Purchase only) — depends on the selected Membership; filtered from the same memberships response

## Why sheet-backed

The Zapier version uses one connected account per gym, with Branch ID / API Key / API Token entered manually each time. With dozens of studios this gets unwieldy. In this n8n version:

- One credential per **n8n instance** (not per gym), pointing at the studio config sheet via a Google Service Account
- The node looks up the selected studio's Branch ID / API Key / API Token from the sheet at execution time
- Adding a new gym = adding a row in the sheet, full stop

## Installation (self-hosted n8n)

Pre-built community packages can be installed from npm via n8n's UI: **Settings → Community Nodes → Install a community node**. Once this package is published to npm, type `n8n-nodes-glofox` and click Install.

Until that publish step, you can install directly from this repo into your self-hosted n8n container. Two options:

### Option A — via Community Nodes UI with a git URL
n8n's recent versions support installing community nodes from a Git URL. Provide:
```
git+https://github.com/SocialFitnessManchester/glofox-n8n-app.git
```

### Option B — manual npm install inside the n8n container
```bash
cd /home/node/.n8n/custom
npm install github:SocialFitnessManchester/glofox-n8n-app
```
Then restart n8n. The node will appear under **Glofox** in the nodes picker.

## Setting up the credential

After installation:

1. **Create a Google Service Account** (or reuse the one already shared with your studio config sheet) — make sure it has at least Viewer access on the sheet.
2. In n8n: **Credentials → New → Glofox (Sheet-backed)**.
3. Fill in:
   - **Studio Config Sheet ID** — long string between `/d/` and `/edit` in the sheet's URL
   - **Sheet Tab Name** — usually `Sheet1`
   - **Service Account Email** — `…@…iam.gserviceaccount.com`
   - **Service Account Private Key** — the PEM block including `-----BEGIN PRIVATE KEY-----` / `-----END PRIVATE KEY-----`. Literal `\n` escape sequences are fine; the node converts them.
4. Save.

## Sheet structure

The first row of the configured sheet/tab must be a header row. The node looks for these columns by name (case-insensitive prefix match, so trailing parenthetical notes are fine):

| Studio Name | Branch ID | API Key | API Token |
|---|---|---|---|

Extra columns are ignored. Rows missing any of those four values are skipped.

## Local development

```bash
# Install dependencies
npm install

# Build TypeScript + copy icon assets
npm run build

# Or watch for changes during development
npm run dev
```

To test against a local n8n, build the package, then `npm link` it from your n8n custom-nodes directory. See [n8n's docs on developing community nodes](https://docs.n8n.io/integrations/creating-nodes/build/) for details.

## Reference

- **Glofox API base:** `https://gf-api.aws.glofox.com/prod`
- **Glofox API docs:** <https://apidocs-plat.aws.glofox.com/flows/lead-sale/>
- **Zapier source this is ported from:** [SocialFitnessManchester/glofox-zapier-app](https://github.com/SocialFitnessManchester/glofox-zapier-app) (private)
