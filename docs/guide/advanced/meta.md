# Route Meta Fields

You can include a `meta` field when defining a route with any arbitrary information:

```js
const routes = [
  {
    path: '/posts',
    component: PostsLayout,
    children: [
      {
        path: 'new',
        component: PostsNew,
        // only authenticated users can create posts
        meta: { requiresAuth: true }
      },
      {
        path: ':id',
        component: PostsDetail
        // anybody can read a post
        meta: { requiresAuth: false }
      }
    ]
  }
]
```

So how do we access this `meta` field?

<!-- TODO: the explanation about route records should be explained before and things should be moved here -->

First, each route object in the `routes` configuration is called a **route record**. Route records may be nested. Therefore when a route is matched, it can potentially match more than one route record.

For example, with the above route config, the URL `/posts/new` will match both the parent route record (`path: '/posts'`) and the child route record (`path: 'new'`).

All route records matched by a route are exposed on the `$route` object (and also route objects in navigation guards) as the `$route.matched` Array. We could loop through that array to check all `meta` fields, but Vue Router also provides you a `$route.meta` that is a non-recursive merge of **all `meta`** fields from parent to child. Meaning you can simply write

```js
router.beforeEach((to, from) => {
  // instead of having to check every route record with
  // to.matched.some(record => record.meta.requiresAuth)
  if (to.meta.requiresAuth && !auth.isLoggedIn()) {
    // this route requires auth, check if logged in
    // if not, redirect to login page.
    return {
      path: '/login',
      // save the location we were at to come back later
      query: { redirect: to.fullPath },
    }
  }
})
```

## TypeScript

It is possible to type the meta field by extending the `RouteMeta` interface:

```ts
declare module 'vue-router' {
  interface RouteMeta {
    // is optional
    isAdmin?: boolean
    // must be declared by every route
    requiresAuth: boolean
  }
}
```