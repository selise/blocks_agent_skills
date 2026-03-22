---
name: blocks-add-database
description: Store and retrieve application data using the SELISE Blocks DataGateway (GraphQL). Use when you need a managed cloud database for your app — no backend required. Covers schema creation in Blocks Cloud, access control, publishing, and making GraphQL queries/mutations from any frontend or backend.
argument-hint: "[x-blocks-key] [project-slug]"
disable-model-invocation: true
---

You are connecting an application to the SELISE Blocks DataGateway. This gives the app managed cloud storage it can query via GraphQL — no backend required.

## What this skill sets up

- A data schema (like a database table) in Blocks Cloud
- Public or authenticated read/write access rules
- A GraphQL endpoint the app can POST to for all data operations
- Working code examples to query and mutate data

---

## Step 1 — Collect credentials

Ask the user to have these ready:

1. **x-blocks-key** — the project API key (found in Blocks Cloud → your project → **Project Settings**, or in the `.env` file as `VITE_X_BLOCKS_KEY`)
2. **project-slug** — the short identifier in the hosted URL, e.g. `dsjogn` from `https://dsjogn-dqnxc.seliseblocks.com`

The DataGateway endpoint is always:
```
https://api.seliseblocks.com/uds/v1/{project-slug}/gateway
```

If the user already provided these in their message, extract them and skip asking.

Store both values. You will use them in every code example in this skill.

---

## Step 2 — Design the schema

Ask the user what data they want to store. For each entity, determine:

- **Schema name**: PascalCase, no spaces (e.g. `TodoItem`, `BlogPost`, `GameScore`)
- **Fields**: name + type for each field the app needs

Supported field types:
| Type | Use for |
|------|---------|
| `String` | Text, IDs, URLs, enums stored as strings |
| `Int` | Whole numbers (counts, scores, ranks) |
| `Float` | Decimal numbers (prices, coordinates) |
| `Boolean` | True/false flags |
| `DateTime` | Timestamps (the schema also auto-provides `CreatedDate`, `LastUpdatedDate`) |

**Note:** Every schema automatically includes these system fields — you do NOT need to define them:
`ItemId`, `CreatedDate`, `CreatedBy`, `LastUpdatedDate`, `LastUpdatedBy`, `IsDeleted`, `Language`, `OrganizationIds`, `Tags`, `DeletedDate`

Present the proposed schema to the user and confirm before proceeding.

---

## Step 3 — Create the schema in Blocks Cloud

> **Prefer to skip the UI?** Use the `blocks-schema-from-curl` sub-skill — paste one authenticated cURL from your browser and the agent calls the API directly to create, configure, and publish the schema for you.

Tell the user to perform these steps manually in their browser:

1. Go to **[https://cloud.seliseblocks.com](https://cloud.seliseblocks.com)** and log in
2. Select the correct **project** and **environment** (top navigation bar)
3. In the left sidebar click **Data** → **Data Gateway**
4. Click **+ Add** to create a new schema
5. Enter the schema name (from Step 2) and select **Entity** as the type
6. Click **Save** — the schema appears in the list
7. Click the schema to open it, then click **Edit**
8. For each field from Step 2: click **+ Add property**, enter the field name and select the type
9. Click **Save** once all fields are added

After saving, confirm with the user that all fields appear correctly before continuing.

---

## Step 4 — Set access control

The DataGateway enforces per-operation access control. The defaults require authentication for everything.

Tell the user to:

1. With the schema selected in Data Gateway, click **Schema Access**
2. A dialog appears with four operations: **View**, **Create**, **Edit**, **Delete**
3. Set access levels according to the app's requirements:

| Scenario | View | Create | Edit | Delete |
|----------|------|--------|------|--------|
| Public leaderboard / read-only content | Public | Logged in users | Logged in users | Logged in users |
| Any user can submit data (no login required) | Public | **Public** | Logged in users | Logged in users |
| Only authenticated users | Logged in users | Logged in users | Logged in users | Logged in users |
| Fully open (prototyping only) | Public | Public | Public | Public |

4. Click **Save** in the dialog

Ask the user what access level they need and confirm the right combination before they apply it.

---

## Step 5 — Publish

Schema changes are staged until published. Tell the user to:

1. Click the **Publish** button (top of the Schemas panel)
2. Wait for the **"Schemas published successfully"** toast notification

The schema is now live and the GraphQL API reflects the new fields.

---

## Step 6 — Verify with the Playground

Before writing any code, confirm the schema works:

1. In Data Gateway, click **Playground** (top right)
2. In the left panel, paste this query (replace `YourSchemaName` with the actual name):

```graphql
query {
  getYourSchemaNames(input: { pageNo: 1, pageSize: 10 }) {
    totalCount
    items {
      ItemId
      CreatedDate
      # add your custom fields here
    }
  }
}
```

3. Click the **Run** button — you should get `{ "data": { "getYourSchemaNames": { "totalCount": 0, "items": [] } } }`
4. An empty result with no errors means everything is wired up correctly

**GraphQL naming rules** (important — the gateway is strict about these):
- Query: `get` + SchemaName + `s` → `getTodoItems`
- Insert mutation: `insert` + SchemaName → `insertTodoItem`
- Update mutation: `update` + SchemaName → `updateTodoItem`
- Delete mutation: `delete` + SchemaName → `deleteTodoItem`
- Input type for insert: SchemaName + `InsertInput` → `TodoItemInsertInput`
- Input type for update: SchemaName + `UpdateInput` → `TodoItemUpdateInput`
- Filter uses MongoDB JSON syntax: `filter: "{\"_id\": \"abc123\"}"`

---

## Step 7 — Add code to the application

Generate working code for the user's tech stack. Use the credentials from Step 1.

### JavaScript / TypeScript (fetch)

```javascript
const GATEWAY_ENDPOINT = 'https://api.seliseblocks.com/uds/v1/{project-slug}/gateway';
const BLOCKS_KEY = '{x-blocks-key}';

// accessToken: optional JWT from the app's auth session
// ALWAYS pass credentials:'omit' — without it, the browser auto-sends session cookies from
// the *.seliseblocks.com domain. The DataGateway then evaluates the cookie token (which is
// the cloud-admin token with aud:cloud.seliseblocks.com) instead of your Bearer header and
// returns 401 or 400. credentials:'omit' prevents cookie interference entirely.
async function gatewayRequest(query, variables = null, accessToken = null) {
  const headers = {
    'Content-Type': 'application/json',
    'x-blocks-key': BLOCKS_KEY,
  };
  if (accessToken) {
    headers['Authorization'] = `Bearer ${accessToken}`;
  }

  const body = variables ? { query, variables } : { query };

  const response = await fetch(GATEWAY_ENDPOINT, {
    method: 'POST',
    headers,
    credentials: 'omit',           // prevents cookie-token interference on *.seliseblocks.com
    referrerPolicy: 'no-referrer', // suppresses Origin header — required for localhost dev
    body: JSON.stringify(body),
  });

  if (!response.ok) {
    const text = await response.text();
    throw new Error(`Gateway error ${response.status}: ${text}`);
  }

  const result = await response.json();

  if (result.errors) {
    throw new Error(result.errors.map(e => e.message).join(', '));
  }

  return result.data;
}

// --- Query example ---
async function listItems() {
  const query = `
    query {
      getYourSchemaNames(input: { pageNo: 1, pageSize: 50 }) {
        totalCount
        items {
          ItemId
          CreatedDate
          FieldOne
          FieldTwo
        }
      }
    }
  `;

  const data = await gatewayRequest(query);
  return data.getYourSchemaNames.items;
}

// --- Insert example ---
async function createItem(fieldOne, fieldTwo) {
  const mutation = `
    mutation($input: YourSchemaNameInsertInput!) {
      insertYourSchemaName(input: $input) {
        itemId
        acknowledged
      }
    }
  `;

  const data = await gatewayRequest(mutation, {
    input: { FieldOne: fieldOne, FieldTwo: fieldTwo },
  });

  return data.insertYourSchemaName.itemId;
}

// --- Update example ---
// IMPORTANT: `filter` is a SEPARATE argument to the mutation, NOT a field inside the input object.
// Putting Filter inside the input will produce: "The field `Filter` does not exist on XUpdateInput"
async function updateItem(itemId, fieldOne) {
  const filter = JSON.stringify(JSON.stringify({ _id: itemId })); // double-stringify for inline GraphQL
  const mutation = `
    mutation($input: YourSchemaNameUpdateInput!) {
      updateYourSchemaName(filter: ${filter}, input: $input) {
        acknowledged
        totalImpactedData
      }
    }
  `;

  const data = await gatewayRequest(mutation, {
    input: { FieldOne: fieldOne },
  });

  return data.updateYourSchemaName.acknowledged;
}

// --- Delete example ---
async function deleteItem(itemId) {
  const mutation = `
    mutation {
      deleteYourSchemaName(filter: "${JSON.stringify({ _id: itemId }).replace(/"/g, '\\"')}") {
        acknowledged
        totalImpactedData
      }
    }
  `;

  const data = await gatewayRequest(mutation);
  return data.deleteYourSchemaName.acknowledged;
}
```

### React with TanStack Query

```typescript
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

// Reuse the gatewayRequest helper from above

export function useItems() {
  return useQuery({
    queryKey: ['yourSchemaNames'],
    queryFn: listItems,
  });
}

export function useCreateItem() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: ({ fieldOne, fieldTwo }) => createItem(fieldOne, fieldTwo),
    onSuccess: () => queryClient.invalidateQueries({ queryKey: ['yourSchemaNames'] }),
  });
}
```

### Node.js / server-side

```javascript
// Node 18+ has native fetch. For older versions: npm install node-fetch
const response = await fetch('https://api.seliseblocks.com/uds/v1/{project-slug}/gateway', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-blocks-key': process.env.BLOCKS_KEY,
    // No referrerPolicy needed server-side — no Origin header is sent
  },
  body: JSON.stringify({ query, variables }),
});
```

### Python

```python
import httpx  # or: import requests

GATEWAY_ENDPOINT = "https://api.seliseblocks.com/uds/v1/{project-slug}/gateway"
BLOCKS_KEY = "{x-blocks-key}"

def gateway_request(query: str, variables: dict = None):
    payload = {"query": query}
    if variables:
        payload["variables"] = variables

    response = httpx.post(
        GATEWAY_ENDPOINT,
        headers={
            "Content-Type": "application/json",
            "x-blocks-key": BLOCKS_KEY,
        },
        json=payload,
    )
    response.raise_for_status()
    result = response.json()

    if "errors" in result:
        raise Exception(result["errors"][0]["message"])

    return result["data"]
```

---

## Step 8 — Sending authenticated requests (optional)

If the schema requires **Logged in users** access, the request must include a JWT.

> **Do NOT rely on the browser auto-sending the Blocks cookie.** When you add `credentials: 'omit'` (required to prevent cookie interference — see Step 7), the browser no longer sends any cookies. Pass the token explicitly in all cases.

The JWT is available in the cookie named `access_token_{x-blocks-key}` after the user logs in via Blocks IDP. For apps that implement their own auth flow, store the token in memory or `localStorage` at login time.

Pass it via the `accessToken` parameter of `gatewayRequest`:

```javascript
const data = await gatewayRequest(mutation, variables, myStoredAccessToken);
```

> **Delete operations always require authentication** — even if `deleteAccessLevel` is set to `2` (Public). The DataGateway returns `AUTH_NOT_AUTHENTICATED` for delete regardless of the access level setting. Always pass a valid Bearer token for delete mutations.

> **Token audience matters**: the token must be scoped to your game/project (`aud: your-slug.seliseblocks.com`). A Blocks Cloud admin token (`aud: cloud.seliseblocks.com`) will be rejected by the DataGateway with a 401. Use only tokens obtained by logging in through your project's IDP endpoint, not from the Blocks Cloud UI session.

---

## Step 9 — CORS and local development

The gateway allows requests only from origins registered with the project.

| Where the app runs | What happens | Fix |
|--------------------|--------------|-----|
| `https://{slug}.seliseblocks.com` | Works automatically — origin is whitelisted | Nothing needed |
| `localhost` / `127.0.0.1` | Browser sends `Origin: http://localhost` → gateway returns **406 Invalid_Origin_Or_Referer** | Add `referrerPolicy: 'no-referrer'` to every fetch call (suppresses the Origin header) |
| Server-side (Node, Python) | No browser CORS — works without any special handling | Nothing needed |

The `referrerPolicy: 'no-referrer'` trick is safe and is the pattern used in the official SELISE reference implementation. It only affects the `Referer` and `Origin` request headers.

---

## Common Mistakes

| Error | Cause | Fix |
|-------|-------|-----|
| `406 Invalid_Origin_Or_Referer` | Browser sending `Origin: http://localhost` | Add `referrerPolicy: 'no-referrer'` |
| `401 Unauthorized` or `400` on any request | Browser cookie from `*.seliseblocks.com` domain sent automatically; DataGateway evaluates the cookie token (cloud-admin, wrong audience) over your Bearer header | Add `credentials: 'omit'` to every fetch call |
| `401 Unauthorized` (empty body) | Schema access is "Logged in users" but no token sent | Pass a valid project-scoped Bearer token |
| `AUTH_NOT_AUTHENTICATED` on delete | Delete always requires auth even with `deleteAccessLevel: 2` | Pass a project-scoped Bearer token for all delete mutations |
| `401` on delete with what looks like a valid token | Token is from Blocks Cloud UI session (`aud: cloud.seliseblocks.com`); DataGateway requires project-scoped token (`aud: your-slug.seliseblocks.com`) | Use a token obtained via your project's own IDP login |
| `400` + `The field 'Filter' does not exist on type XUpdateInput` | Filter was placed inside the input object | Move filter to a separate mutation argument: `updateX(filter: "...", input: $input)` |
| `400` + `field X does not exist on type YInsertInput` | Field not in schema, or schema not published after adding it | Add field in Blocks Cloud → Save → Publish |
| `400` + `Cannot query field X` | Field name typo or wrong schema name in query | Check exact names in Blocks Cloud → Data Gateway |
| `400` + `Variable "$input" of required type "XInsertInput!" was not provided` | Missing `variables` in the fetch body | Ensure `body: JSON.stringify({ query, variables })` |
| `200` but `result.data` is null | GraphQL error — check `result.errors[0].message` | Log full response before parsing |
| Bulk delete only removes one record | `deleteX(filter: "{}")` deletes exactly one document per call regardless of filter | Loop the delete call N times (once per record); fetch totalCount first to know N |

---

## Done

At the end of this skill the user should have:

- [ ] A published schema with the right fields
- [ ] Access control set correctly for their use case
- [ ] A working `gatewayRequest` helper in their codebase
- [ ] At least one query and one mutation tested against real data

If anything fails, re-read the error from the response body — the DataGateway always returns a descriptive message in `errors[0].message`.
