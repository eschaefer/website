---
title: 'Import all modules in a directory with webpack'
tags: [til, javascript, webpack]
published: true
comments: false
---

Today I needed to import any number of markdown files in a directory, and then render them in a list in a React app. If this was in a node.js environment, we could do this with `fs.readdir` and easily get the list of files and resolve them. However, this is not possible when running Javascript in the browser. So then how can I do this?

Since my app is using Webpack as a bundler, I started digging around the [documentation](https://webpack.js.org/guides/dependency-management/#requirecontext) and found `require.context()`.

> It allows you to pass in a directory to search, a flag indicating whether subdirectories should be searched too, and a regular expression to match files against.

Exactly what I need!

Here's what I came up with:

```typescript
async function importAll(r: __WebpackModuleApi.RequireContext) {
  const keys = r
    .keys()
    .filter((key) => key.startsWith('./'))
    .map((key) => key.replace('./', ''));

  const importPromises = keys.map(async (key) => {
    return await import(`../../content/changelog/${key}`);
  });

  try {
    const resolved = await Promise.all(importPromises);
    return resolved;
  } catch (error) {
    console.error(error);
  }
}

const allChangelogs = require.context(
  '../../content/changelog/',
  false,
  /\.md$/
);

importAll(allChangelogs).then((modules) => {
  if (!modules) throw new Error('Missing changelogs');

  // Do something with resolved modules...
});
```
