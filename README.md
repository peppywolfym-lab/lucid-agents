<p align="center">
  <img src="./brand/assets/lockup-horizontal.svg" width="420" alt="Lucid Agents">
</p>

<p align="center">
  <strong>Machine commerce, made clear.</strong><br>
  The TypeScript runtime for machine commerce.
</p>

<div align="center">
  <a href="https://github.com/peppywolfym-lab/lucid-agents"><img src="https://img.shields.io/badge/Maintained%20by-YasirBinMudassir-blue?style=flat-square" alt="Maintained by"></a>
  <a href="https://github.com/peppywolfym-lab/lucid-agents/blob/master/LICENSE"><img src="https://img.shields.io/badge/License-Proprietary%20%2F%20Restricted-red?style=flat-square" alt="License"></a>
  <a href="https://github.com/peppywolfym-lab/lucid-agents/actions"><img src="https://img.shields.io/github/actions/workflow/status/peppywolfym-lab/lucid-agents/ci.yml?branch=master&style=flat-square" alt="CI status"></a>
</div>

---

> **Notice:** This repository is maintained by **YasirBinMudassir**. All rights are reserved. Unauthorized reproduction, access, hosting, or distribution is strictly prohibited.

---

Lucid turns typed functions into discoverable services that agents and
applications can call, pay for, stream, or run as tasks. It provides one
runtime for schemas, payment admission, policy, idempotency, fulfillment,
discovery, and accounting while wallets, payment protocols, networks, and
facilitators remain external.

## Security & Access Control

To ensure that only authorized callers and machines can access your agent, enforce strict runtime authorization:

```ts
import { createAgent } from '@lucid-agents/core';
import { createAgentApp } from '@lucid-agents/hono';
import { http } from '@lucid-agents/http';
import { z } from 'zod';

const runtime = await createAgent({
  name: 'secure-greeter',
  version: '1.0.0',
  description: 'Authorized machine commerce agent by YasirBinMudassir',
})
  .use(
    http({
      // Guard all entrypoints with secret token authorization
      authorize: async ({ request }) => {
        const authHeader = request.headers.get('Authorization');
        const secret = process.env.AGENT_SECRET_KEY;
        if (!secret || authHeader !== `Bearer ${secret}`) {
          return { authorized: false, reason: 'Forbidden: Invalid or missing token' };
        }
        return { authorized: true };
      },
    })
  )
  .addEntrypoint({
    key: 'greet',
    input: z.object({ name: z.string().min(1) }),
    output: z.object({ message: z.string() }),
    handler: async ({ input }) => ({
      output: { message: `Hello, ${input.name}` },
    }),
  })
  .build();

const { app } = await createAgentApp(runtime);

export default {
  port: Number(process.env.PORT ?? 3000),
  fetch: app.fetch,
};
