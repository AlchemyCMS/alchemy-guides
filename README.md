[![Netlify Status](https://api.netlify.com/api/v1/badges/6a12b87f-d5a1-4b15-b723-112daf331c5a/deploy-status)](https://app.netlify.com/sites/alchemy-guides/deploys)

# The AlchemyCMS Guides

The guides are created with [VitePress](https://vitepress.dev/) and are written in [Markdown](https://daringfireball.net/projects/markdown/syntax).

[![Deploys by Netlify](https://www.netlify.com/img/global/badges/netlify-color-bg.svg)](https://www.netlify.com)

## File locations

The actual guides sourcefiles live in the `/guides` folder.

## Preview the guides

Run

```bash
bun install
```

to install [VitePress](https://vitepress.dev/).

To run a local preview server, run

```bash
bun run guides:preview
```

The server will be available at [http://localhost:8080](http://localhost:8080).

## Building

Build the guides html files with:

```bash
bun run guides:build
```

## Deployment

Deployment is done automatically via [netlify](https://www.netlify.com) whenever a change is merged into the main branch.

## Contributing

1. Fork it
2. Create a branch (`git checkout -b my-new-guide`)
3. Commit your changes (`git commit -am 'Add new guide for elements'`)
4. Push to the branch (`git push origin my-new-guide`)
5. Create new Pull Request
