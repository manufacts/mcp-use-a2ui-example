# Generative UI with A2UI

Ask for a UI in chat. The model calls `render-ui` with a layout and initial data,
and the stock **A2UI React renderer** displays it inside an mcp-use MCP App.
The model chooses the components and their arrangement on each call.

This follows the same pattern as the JSON-Render example: one rendering tool,
a small component catalog, and one view. A2UI supplies the renderer and local
data bindings. The server needs no model API key or agent runtime; the host chat
model generates the tool arguments.

## Run

Requires Node.js 22.22.2 or later.

```sh
git clone https://github.com/manufacts/mcp-use-a2ui-example.git
cd mcp-use-a2ui-example
npm ci
npm run dev -- --port 3012
```

Open the printed Inspector URL, select **Chat**, and sign in with Manufact or
configure a supported model provider. Try:

> Make an interactive weekend reading dashboard with two book cards, a
> reading-goal slider, and a checklist. Let me edit the reader name.

Or:

> Build a packing checklist for a three-day beach trip, grouped into essentials
> and clothes, with an editable destination and a travel-style selector.

The app appears when the tool response completes. Inputs, checkboxes, sliders,
and choices edit local state. Components sharing a `/key` binding update
together. Edits are not saved and reset when the app is recreated; this example
does not implement submission, calculations, or server actions.

## Three pieces

- `src/index.ts`: the `render-ui` tool accepts and returns `{ spec }`.
- `views/generative-ui/catalog.ts`: the supported components and validation for
  IDs, references, bindings, and initial values.
- `views/generative-ui/view.tsx`: turn the spec into A2UI messages and render
  `<A2uiSurface>`. `view.css` supplies a small theme.

The catalog supports Text, Row, Column, Card, Divider, TextField, CheckBox,
Slider, and ChoicePicker. To extend it, add the matching A2UI component shape
to the schema and describe its use in the tool instructions.

The view passes standard A2UI v0.9 `createSurface`, `updateComponents`, and
`updateDataModel` messages to the processor. Everything travels through a normal
MCP tool response and MCP App; no AG-UI endpoint or first-party SDK adapter is
needed. A2UI's internal dependencies use Zod 3, while the tool schema uses Zod 4
for JSON Schema generation.

## Connect CopilotKit

Use CopilotKit's MCP Apps middleware with the public deployment:

```ts
import { MCPAppsMiddleware } from "@ag-ui/mcp-apps-middleware";

agent.use(new MCPAppsMiddleware({
  mcpServers: [{
    type: "http",
    url: "https://warm-steel-y2d5o.run.mcp-use.com/mcp",
    serverId: "mcp-use-a2ui",
  }],
}));
```

Keep your existing agent's model configuration and register it with your
CopilotRuntime. CopilotKit's built-in MCP Apps renderer loads the view in chat.
For a local runtime, use `http://localhost:3012/mcp`. A hosted runtime needs the
public URL. No A2UI catalog registration is required in the host: this MCP App
contains its own A2UI renderer.

See [CopilotKit's MCP Apps guide](https://docs.copilotkit.ai/generative-ui/mcp-apps).
The deployed tool and resource endpoints have been smoke-tested; end-to-end
CopilotKit chat validation is pending host configuration.

## Check and build

```sh
npm run typecheck
npm run build
```

References: [A2UI React renderer](https://github.com/a2ui-project/a2ui/tree/main/renderers/react),
[A2UI inside MCP Apps](https://a2ui.org/guides/a2ui-in-mcp-apps/).

## Deploy

Build with `npm ci && npm run build` and start with `npm start`. The MCP endpoint is `/mcp`. No model API key is required on this server; the connected chat host supplies the model.

Source: [mcp-use A2UI example](https://github.com/mcp-use/mcp-use/tree/codex/a2ui-example/libraries/typescript/packages/server/examples/views/a2ui).
