---
title: 'Next.js: Unexpected missing ctx.req in _document'
tags: [javascript, til]
published: true
comments: false
---

This week I was trying to access the `ctx.req` property on a `_document`, but the value kept showing up as `undefined`. I tried the most basic example that [they showed in their docs](https://nextjs.org/docs/advanced-features/custom-document):

```javascript
class MyDocument extends Document {
  static async getInitialProps(ctx) {
    const initialProps = await Document.getInitialProps(ctx);

    // Expecting headers, cookies, etc, but it's undefined.
    console.log(ctx.req);

    return initialProps;
  }

  render() {
    return (
      <Html>
        <Head />
        <body>
          <Main />
          <NextScript />
        </body>
      </Html>
    );
  }
}

export default MyDocument;
```

Why is `ctx.req` undefined on a server-side only process? The Next.js docs say about the custom `_document`:

> This file is only rendered on the server

💡 Well, it turns out that _another_ page rendered lower in the tree by `<Main />` must invoke `getServerSideProps` first. For example, adding this to `pages/index.tsx`:

```typescript
export const getServerSideProps = async (_ctx: GetServerSidePropsContext) => {
  /**
   * Yep, it's empty, but needed to force reading cookies and request
   * headers on server-side so _document can read it
   */
  return { props: {} };
};
```

Yep, and now all of a sudden, the full request object is available on `ctx.req` inside the custom `_document`!

It matters which route/page you're accessing. So, adding `getServerSideProps` to `pages/index.tsx` won't mean that `_document` will behave the same on `pages/login.tsx`. You'd need to add `getServerSideProps` on that page as well.
