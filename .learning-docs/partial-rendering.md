10

Chapter 10

# Partial Prerendering (Optional)

> Partial Prerendering is an experimental feature introduced in Next.js 14. The content of this page may be updated as the feature progresses in stability. You may want to skip this chapter if you prefer to not use experimental features. This chapter is not required to complete the course.

In this chapter...

Here are the topics we’ll cover

What Partial Prerendering is.

How Partial Prerendering works.

## [Combining Static and Dynamic Content](https://nextjs.org/learn/dashboard-app/partial-prerendering#combining-static-and-dynamic-content)

Currently, if you call a [dynamic function](https://nextjs.org/docs/app/building-your-application/routing/route-handlers#dynamic-functions) inside your route (e.g. `noStore()`, `cookies()`, etc), your whole route becomes dynamic.

This aligns with how most web apps are built today, you either choose between static and dynamic rendering for your **entire application** or for **specific routes**.

However, most routes are not fully static or dynamic. You may have a route that has both static and dynamic content. For example, let's say you have a social media feed, the posts would be static, but the likes for the post would be dynamic. Or an ecommerce site, where the product details are static, but the user's cart is dynamic.

Going back to your dashboard page, what components would you consider static vs. dynamic?

Once you're ready, click the button below to see how we would split the dashboard route:

Hide the solution

![Diagram showing how the sidenav is static while page's children are dynamic|1600x566](https://nextjs.org/_next/image?url=%2Flearn%2Fdark%2Fdashboard-static-dynamic-components.png&w=3840&q=75&dpl=dpl_7TyN5UAPRwBMqu696tYhRxfarEr1)

- The `<SideNav>` Component doesn't rely on data, and is not personalized to the user, so it can be **static**.
- The components in `<Page>` rely on data that changes often and will be personalized to the user, so they can be **dynamic**.

## [What is Partial Prerendering?](https://nextjs.org/learn/dashboard-app/partial-prerendering#what-is-partial-prerendering)

In Next.js 14, there is a preview of a new rendering model called **Partial Prerendering**. Partial Prerendering is an experimental feature that allows you to render a route with a static loading shell, while keeping some parts dynamic. In other words, you can isolate the dynamic parts of a route. For example:

![Partially Prerendered Product Page showing static nav and product information, and dynamic cart and recommended products|1600x632](https://nextjs.org/_next/image?url=%2Flearn%2Fdark%2Fthinking-in-ppr.png&w=3840&q=75&dpl=dpl_7TyN5UAPRwBMqu696tYhRxfarEr1)

When a user visits a route:

- A static route _shell_ is served, this makes the initial load fast.
- The shell leaves _holes_ where dynamic content will load in async.
- The async holes are loaded in parallel, reducing the overall load time of the page.

This is different from how your application behaves today, where entire routes are either fully static or dynamic.

Partial Prerendering combines ultra-quick static edge delivery with fully dynamic capabilities and we believe it has the potential to [become the default rendering model for web applications](https://vercel.com/blog/partial-prerendering-with-next-js-creating-a-new-default-rendering-model), bringing together the best of static site generation and dynamic delivery.

### It’s time to take a quiz!

Test your knowledge and see what you’ve just learned.

What are the holes in the context of Partial Prerendering?

A

Locations where JavaScript is disabled

B

Locations where dynamic content will load asynchronously

C

Locations where third-party scripts are loaded

Check Answer

## [How does Partial Prerendering work?](https://nextjs.org/learn/dashboard-app/partial-prerendering#how-does-partial-prerendering-work)

Partial Prerendering leverages React's [Concurrent APIs](https://react.dev/blog/2021/12/17/react-conf-2021-recap#react-18-and-concurrent-features) and uses [Suspense](https://react.dev/reference/react/Suspense) to defer rendering parts of your application until some condition is met (e.g. data is loaded).

The fallback is embedded into the initial static file along with other static content. At build time (or during revalidation), the static parts of the route are _prerendered_, and the rest is _postponed_ until the user requests the route.

It's worth noting that wrapping a component in Suspense doesn't make the component itself dynamic (remember you used `unstable_noStore` to achieve this behavior), but rather Suspense is used as a boundary between the static and dynamic parts of your route.

The great thing about Partial Prerendering is that you don't need to change your code to use it. As long as you're using Suspense to wrap the dynamic parts of your route, Next.js will know which parts of your route are static and which are dynamic.

> **Note:** To learn more about how Partial Prerendering can be configured, see the [Partial Prerendering (experimental) documentation](https://nextjs.org/docs/app/api-reference/next-config-js/partial-prerendering) or try the [Partial Prerendering template and demo](https://vercel.com/templates/next.js/partial-prerendering-nextjs). It's important to note that this feature is **experimental** and **not yet ready for production deployment**.

## [Summary](https://nextjs.org/learn/dashboard-app/partial-prerendering#summary)

To recap, you've done a few things to optimize data fetching in your application, you've:

1. Created a database in the same region as your application code to reduce latency between your server and database.
2. Fetched data on the server with React Server Components. This allows you to keep expensive data fetches and logic on the server, reduces the client-side JavaScript bundle, and prevents your database secrets from being exposed to the client.
3. Used SQL to only fetch the data you needed, reducing the amount of data transferred for each request and the amount of JavaScript needed to transform the data in-memory.
4. Parallelize data fetching with JavaScript - where it made sense to do so.
5. Implemented Streaming to prevent slow data requests from blocking your whole page, and to allow the user to start interacting with the UI without waiting for everything to load.
6. Move data fetching down to the components that need it, thus isolating which parts of your routes should be dynamic in preparation for Partial Prerendering.

In the next chapter, we'll look at two common patterns you might need to implement when fetching data: search and pagination.

10

## You've Completed Chapter 10

You've learned about Partial Prerendering, an early rendering model introduced in Next.js 14.

Next Up

11: Adding Search and Pagination

Learn how to implement search and pagination with Next.js APIs.

[Start Chapter 11](./adding-search-and-pagination.md)
