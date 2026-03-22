# Blocks DataGateway: GraphQL Implementation & Troubleshooting

This Knowledge Item summarizes hard-won engineering context for integrating applications with the SELISE Blocks DataGateway.

## 1. Endpoint Architecture
The DataGateway provides a unified GraphQL endpoint for all project schemas.
- **Production Standard**: `https://api.seliseblocks.com/uds/v1/{project-slug}/gateway`
- **Internal/Legacy**: `https://{tenantId}.seliseblocks.com/datagateway/graphql` (Avoid unless requested)

> [!IMPORTANT]
> Always verify the **project-slug** (e.g., `dsjogn`) from the hosted URL or project settings.

## 2. Strict Naming Conventions
The DataGateway auto-generates operations based on the **SchemaName** (PascalCase).Typos here result in `400 Bad Request` or `Cannot query field...` errors.

| Operation | Format | Example (Schema: `GameScore`) |
|-----------|--------|------------------------------|
| **Query** | `get` + SchemaName + `s` | `getGameScores` |
| **Insert** | `insert` + SchemaName | `insertGameScore` |
| **Update** | `update` + SchemaName | `updateGameScore` |
| **Delete** | `delete` + SchemaName | `deleteGameScore` |

### Input Types
- **Insert**: `GameScoreInsertInput`
- **Update**: `GameScoreUpdateInput`

## 3. Mandatory Request Headers
To avoid CORS and authentication interference:
```javascript
{
  'Content-Type': 'application/json',
  'x-blocks-key': BLOCKS_PROJECT_KEY,
  'Authorization': `Bearer ${accessToken}` // Required for Delete & Authenticated Schemas
}
```

## 4. Common Troubleshooting Patterns

### 401 Unauthorized (The 'Audience' Trap)
- **Problem**: Passing a token obtained from the Blocks Cloud UI (Blocks Central).
- **Detail**: Blocks Cloud tokens have `aud: cloud.seliseblocks.com`. The DataGateway requires a project-scoped audience: `aud: {project-slug}.seliseblocks.com`.
- **Fix**: Log in via the project's own IDP endpoint (`https://api.seliseblocks.com/idp/v1/{tenantId}/token`) to get a valid token.

### 401/400 (Cookie Interference)
- **Problem**: Browser sends `*.seliseblocks.com` cookies automatically. The Gateway prioritizes these over your Bearer header.
- **Fix**: Always set `credentials: 'omit'` in the `fetch` options.

### 406 Invalid Origin/Referrer
- **Problem**: Localhost development triggers CORS security blocks.
- **Fix**: Set `referrerPolicy: 'no-referrer'` in the `fetch` options to suppress the `Origin` header.

### Deleting Multiple Records
- **Problem**: `deleteSchemaName(filter: "{}")` only deletes **one** document per call.
- **Fix**: Fetch the `totalCount` first, then loop the delete mutation N times.

## 5. Knowledge Discovery Checklist
Before starting a Blocks task:
1. **Check KI Summaries**: Search for "Blocks", "DataGateway", or "IDP".
2. **Review `/blocks_agent_skills`**: The `blocks-add-database` skill contains the most up-to-date endpoint and naming logic.
3. **Verify Tenant and Slug**: Ensure the `tenantId` (UUID) and `project-slug` (Short name) are not mixed up in paths.
