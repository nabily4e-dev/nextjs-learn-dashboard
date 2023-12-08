5

Chapter 5

# Navigating Between Pages

In the previous chapter, you created the dashboard layout and pages. Now, let's add some links to allow users to navigate between the dashboard routes.

In this chapter...

Here are the topics we’ll cover

How to use the `next/link` component.

How to show an active link with the `usePathname()` hook.

How navigation works in Next.js.

## [Why optimize navigation?](https://nextjs.org/learn/dashboard-app/navigating-between-pages#why-optimize-navigation)

To link between pages, you'd traditionally use the `<a>` HTML element. At the moment, the sidebar links use `<a>` elements, but notice what happens when you navigate between the home, invoices, and customers pages on your browser.

Did you see it?

There's a full page refresh on each page navigation!

## [The `<Link>` component](https://nextjs.org/learn/dashboard-app/navigating-between-pages#the-link-component)

In Next.js, you can use the `<Link />` Component to link between pages in your application. `<Link>` allows you to do [client-side navigation](https://nextjs.org/docs/app/building-your-application/routing/linking-and-navigating#how-routing-and-navigation-works) with JavaScript.

To use the `<Link />` component, open `/app/ui/dashboard/nav-links.tsx`, and import the `Link` component from [`next/link`](https://nextjs.org/docs/app/api-reference/components/link). Then, replace the `<a>` tag with `<Link>`:

/app/ui/dashboard/nav-links.tsx

```
import {  UserGroupIcon,  HomeIcon,  DocumentDuplicateIcon,} from '@heroicons/react/24/outline';import Link from 'next/link'; // ... export default function NavLinks() {  return (    <>      {links.map((link) => {        const LinkIcon = link.icon;        return (          <Link            key={link.name}            href={link.href}            className="flex h-[48px] grow items-center justify-center gap-2 rounded-md bg-gray-50 p-3 text-sm font-medium hover:bg-sky-100 hover:text-blue-600 md:flex-none md:justify-start md:p-2 md:px-3"          >            <LinkIcon className="w-6" />            <p className="hidden md:block">{link.name}</p>          </Link>        );      })}    </>  );}
```

As you can see, the `Link` component is similar to using `<a>` tags, but instead of `<a href="…">`, you use `<Link href="…">`.

Save your changes and check to see if it works in your localhost. You should now be able to navigate between the pages without seeing a full refresh. Although parts of your application are rendered on the server, there's no full page refresh, making it feel like a web app. Why is that?

### [Automatic code-splitting and prefetching](https://nextjs.org/learn/dashboard-app/navigating-between-pages#automatic-code-splitting-and-prefetching)

To improve the navigation experience, Next.js automatically code splits your application by route segments. This is different from a traditional React [SPA](https://developer.mozilla.org/en-US/docs/Glossary/SPA), where the browser loads all your application code on initial load.

Splitting code by routes means that pages become isolated. If a certain page throws an error, the rest of the application will still work.

Futhermore, in production, whenever [`<Link>`](https://nextjs.org/docs/api-reference/next/link) components appear in the browser's viewport, Next.js automatically **prefetches** the code for the linked route in the background. By the time the user clicks the link, the code for the destination page will already be loaded in the background, and this is what makes the page transition near-instant!

Learn more about [how navigation works](https://nextjs.org/docs/app/building-your-application/routing/linking-and-navigating#how-routing-and-navigation-works).

### It’s time to take a quiz!

Test your knowledge and see what you’ve just learned.

What does Next.js do when a <Link> component appears in the browser’s viewport in a production environment?

A

Prefetches the code for the linked route

Correct

Next.js automatically prefetches the code for the linked route in the background. By the time the user clicks the link, the code for the destination page will already be loaded in the background, and this is what makes the page transition near-instant!

## [Pattern: Showing active links](https://nextjs.org/learn/dashboard-app/navigating-between-pages#pattern-showing-active-links)

A common UI pattern is to show an active link to indicate to the user what page they are currently on. To do this, you need to get the user's current path from the URL. Next.js provides a hook called [`usePathname()`](https://nextjs.org/docs/app/api-reference/functions/use-pathname) that you can use to check the path and implement this pattern.

Since [`usePathname()` ](https://nextjs.org/docs/app/api-reference/functions/use-pathname) is a hook, you'll need to turn `nav-links.tsx` into a Client Component. Add React's `"use client"` directive to the top of the file, then import `usePathname()` from `next/navigation`:

/app/ui/dashboard/nav-links.tsx

```
'use client'; import {  UserGroupIcon,  HomeIcon,  InboxIcon,} from '@heroicons/react/24/outline';import Link from 'next/link';import { usePathname } from 'next/navigation'; // ...
```

Next, assign the path to a variable called `pathname` inside your `<NavLinks />` component:

/app/ui/dashboard/nav-links.tsx

```
export default function NavLinks() {  const pathname = usePathname();  // ...}
```

You can use the `clsx` library introduced in the chapter on [CSS styling](https://nextjs.org/learn/dashboard-app/css-styling) to conditionally apply class names when the link is active. When `link.href` matches the `pathname`, the link should displayed with blue text and a light blue background.

Here's the final code for `nav-links.tsx`:

/app/ui/dashboard/nav-links.tsx

```
'use client'; import {  UserGroupIcon,  HomeIcon,  DocumentDuplicateIcon,} from '@heroicons/react/24/outline';import Link from 'next/link';import { usePathname } from 'next/navigation';import clsx from 'clsx'; // ... export default function NavLinks() {  const pathname = usePathname();   return (    <>      {links.map((link) => {        const LinkIcon = link.icon;        return (          <Link            key={link.name}            href={link.href}            className={clsx(              'flex h-[48px] grow items-center justify-center gap-2 rounded-md bg-gray-50 p-3 text-sm font-medium hover:bg-sky-100 hover:text-blue-600 md:flex-none md:justify-start md:p-2 md:px-3',              {                'bg-sky-100 text-blue-600': pathname === link.href,              },            )}          >            <LinkIcon className="w-6" />            <p className="hidden md:block">{link.name}</p>          </Link>        );      })}    </>  );}
```

Save and check your localhost. You should now see the active link highlighted in blue.

5

## You've Completed Chapter 5

You've learned how to link between pages and leverage client-side navigation in Next.js.

Next Up

6: Setting Up Your Database

Let's create a database to start fetching real data!

[Start Chapter 6](./setting-up-your-database.md)
