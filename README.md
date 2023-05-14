# test-nextjs-withts-chrome-extension

- nextjs, typescript로 chrome-extension 개발 테스트
- popup : nextjs (typescript)
- background, content : typescript

## How to create this project

- `npx create-next-app@latest --experimental`
  - project name : test-nextjs-withts-chrome-extension
  - option
    - Typescript : Yes
    - ESLint : No
    - Tailwind CSS : No
    - src/ directory : Yes
    - App Router : No
    - import alias : No
- `cd test-nextjs-withts-chrome-extension`
- installing packages
  - `npm install --save-dev webpack webpack-cli`
  - `npm install --save-dev copy-webpack-plugin`
  - `npm install --save-dev ts-loader`
  - `npm install --save-dev @types/chrome`
- create `src/extensions` directory
- create `src/extensions/background.ts`
  - `console.log("background script loaded");`
- create `src/extensions/content.ts`
  - `console.log("contents test script");`
- create `manifest.json`
<details>
  <summary> mainfest.json </summary>

```
{
  "name": "NextJS, Typescript Chrome Extension Test",
  "description": "just for test",
  "version": "1.0",
  "manifest_version": 3,
  "permissions": ["activeTab", "scripting"],
  "action": {
    "default_popup": "index.html",
    "default_icon": "favicon.ico"
  },
  "background": {
    "service_worker": "extension/background.js"
  },
  "content_scripts": [
    {
      "js": ["extension/content.js"],
      "matches": ["https://*/*"]
    }
  ]
}
```

</details>

- create `webpack.config.js`
<details>
  <summary> webpack.config.js </summary>

```
const path = require("path");
const CopyPlugin = require("copy-webpack-plugin");
module.exports = {
  mode: "production",
  entry: {
    background: path.resolve(__dirname, "src", "extensions", "background.ts"),
    content: path.resolve(__dirname, "src", "extensions", "content.ts"),
  },
  output: {
    path: path.join(__dirname, "./out/extension"),
    filename: "[name].js",
  },
  resolve: {
    extensions: [".ts", ".js"],
  },
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        loader: "ts-loader",
        exclude: /node_modules/,
      },
    ],
  },
  plugins: [
    new CopyPlugin({
      patterns: [{ from: "manifest.json", to: "../manifest.json" }],
    }),
  ],
};
```

</details>

- modify `package.json`
  - `build` : run `build:next` and then run `build:webpack`
  - `build:next` : build nextjs project and export to `./out` directory. then move `./out/_next` to `./out/next`.
  - `build:webpack` : move `manifest.json`, `background.js`, `content.js` to `./out` directory.

```
{
  ...
  "scripts": {
    ...
    "build": "npm run build:next && npm run build:webpack",
    "build:next": "next build && next export && mv ./out/_next ./out/next && cd ./out && grep -rli '_next' * | xargs -I@ sed -i '' 's|/_next|/next|g' @;",
    "build:webpack": "webpack --config webpack.config.js"
  },
  ...
}
```

<details>
  <summary> trouble shootings (depends on version) </summary>

- modify `next.config.js` (add `images.unoptimized=true`)

```
/** @type {import('next').NextConfig} */
const nextConfig = {
  reactStrictMode: true,
  images: {
    unoptimized: true,
  },
};

module.exports = nextConfig;
```

- modify `tsconfig.json`

  - modify `compilerOptions.noEmit` to `false`

- modify `src/styles/globals.css`
  - add `html, body { max-width: 500 }`

</details>

- `npm run build`
- import `out` directory to chrome://extensions

## For development

## References

---

This is a [Next.js](https://nextjs.org/) project bootstrapped with [`create-next-app`](https://github.com/vercel/next.js/tree/canary/packages/create-next-app).

## Getting Started

First, run the development server:

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
```

Open [http://localhost:3000](http://localhost:3000) with your browser to see the result.

You can start editing the page by modifying `pages/index.tsx`. The page auto-updates as you edit the file.

[API routes](https://nextjs.org/docs/api-routes/introduction) can be accessed on [http://localhost:3000/api/hello](http://localhost:3000/api/hello). This endpoint can be edited in `pages/api/hello.ts`.

The `pages/api` directory is mapped to `/api/*`. Files in this directory are treated as [API routes](https://nextjs.org/docs/api-routes/introduction) instead of React pages.

This project uses [`next/font`](https://nextjs.org/docs/basic-features/font-optimization) to automatically optimize and load Inter, a custom Google Font.

## Learn More

To learn more about Next.js, take a look at the following resources:

- [Next.js Documentation](https://nextjs.org/docs) - learn about Next.js features and API.
- [Learn Next.js](https://nextjs.org/learn) - an interactive Next.js tutorial.

You can check out [the Next.js GitHub repository](https://github.com/vercel/next.js/) - your feedback and contributions are welcome!

## Deploy on Vercel

The easiest way to deploy your Next.js app is to use the [Vercel Platform](https://vercel.com/new?utm_medium=default-template&filter=next.js&utm_source=create-next-app&utm_campaign=create-next-app-readme) from the creators of Next.js.

Check out our [Next.js deployment documentation](https://nextjs.org/docs/deployment) for more details.
