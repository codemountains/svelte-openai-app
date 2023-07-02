# Svelte OpenAI App

OpenAI GPT-3.5-Turbo bot chat demo application by using Svelte and SvelteKit.

> ⚠️ Important note: [openai](https://www.npmjs.com/package/openai?activeTab=readme) is meant for server-side usage only, as using it in client-side browser code will expose your secret API key. See here for more details.

## Developing

Create `src/.env`.

```
PUBLIC_OPENAI_ORGANIZATION_ID = id
PUBLIC_OPENAI_API_KEY = key
```

Once you've created a project and installed dependencies with `npm install` (or `pnpm install` or `yarn`), start a development server:

```bash
npm run dev

# or start the server and open the app in a new browser tab
npm run dev -- --open
```

## Building

To create a production version of your app:

```bash
npm run build
```

You can preview the production build with `npm run preview`.

> To deploy your app, you may need to install an [adapter](https://kit.svelte.dev/docs/adapters) for your target environment.
