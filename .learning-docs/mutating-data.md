12

Chapter 12

# Mutating Data

In the previous chapter, you implemented search and pagination using URL Search Params and Next.js APIs. Let's continue working on the Invoices page by adding the ability to create, update, and delete invoices!

In this chapter...

Here are the topics we’ll cover

What React Server Actions are and how to use them to mutate data.

How to work with forms and Server Components.

Best practices for working with the native `formData` object, including type validation.

How to revalidate the client cache using the `revalidatePath` API.

How to create dynamic route segments with specific IDs.

How to use the React’s `useFormStatus` hook for optimistic updates.

## [What are Server Actions?](https://nextjs.org/learn/dashboard-app/mutating-data#what-are-server-actions)

React Server Actions allow you to run asynchronous code directly on the server. They eliminate the need to create API endpoints to mutate your data. Instead, you write asynchronous functions that execute on the server and can be invoked from your Client or Server Components.

Security is a top priority for web applications, as they can be vulnerable to various threats. This is where Server Actions come in. They offer an effective security solution, protecting against different types of attacks, securing your data, and ensuring authorized access. Server Actions achieve this through techniques like POST requests, encrypted closures, strict input checks, error message hashing, and host restrictions, all working together to significantly enhance your app's safety.

## [Using forms with Server Actions](https://nextjs.org/learn/dashboard-app/mutating-data#using-forms-with-server-actions)

In React, you can use the `action` attribute in the `<form>` element to invoke actions. The action will automatically receive the native [FormData](https://developer.mozilla.org/en-US/docs/Web/API/FormData) object, containing the captured data.

For example:

```tsx
// Server Component
export default function Page() {
  // Action
  async function create(formData: FormData) {
    'use server';
    // Logic to mutate data...
  }
  // Invoke the action using the "action" attribute
  return <form action={create}>...</form>;
}
```

An advantage of invoking a Server Action within a Server Component is progressive enhancement - forms work even if JavaScript is disabled on the client.

## [Next.js with Server Actions](https://nextjs.org/learn/dashboard-app/mutating-data#nextjs-with-server-actions)

Server Actions are also deeply integrated with Next.js [caching](https://nextjs.org/docs/app/building-your-application/caching). When a form is submitted through a Server Action, not only can you use the action to mutate data, but you can also revalidate the associated cache using APIs like `revalidatePath` and `revalidateTag`.

### It’s time to take a quiz!

Test your knowledge and see what you’ve just learned.

What's one benefit of using a Server Actions?

A

Improved SEO.

B

Progressive Enhancement.

C

Faster Websites.

D

Data Encryption.

Check Answer

Let's see how it all works together!

## [Creating an invoice](https://nextjs.org/learn/dashboard-app/mutating-data#creating-an-invoice)

Here are the steps you'll take to create a new invoice:

1. Create a form to capture the user's input.
2. Create a Server Action and invoke it from the form.
3. Inside your Server Action, extract the data from the `formData` object.
4. Validate and prepare the data to be inserted into your database.
5. Insert the data and handle any errors.
6. Revalidate the cache and redirect the user back to invoices page.

### [1. Create a new route and form](https://nextjs.org/learn/dashboard-app/mutating-data#1-create-a-new-route-and-form)

To start, inside the `/invoices` folder, add a new route segment called `/create` with a `page.tsx` file:

![Invoices folder with a nested create folder, and a page.tsx file inside it|1600x363](https://nextjs.org/_next/image?url=%2Flearn%2Fdark%2Fcreate-invoice-route.png&w=3840&q=75&dpl=dpl_9VWEnshqwJ4TWYcNesdqAEpT4iWx)

You'll be using this route to create new invoices. Inside your `page.tsx` file, paste the following code, then spend some time studying it:

/dashboard/invoices/create/page.tsx

```tsx
import Form from '@/app/ui/invoices/create-form';
import Breadcrumbs from '@/app/ui/invoices/breadcrumbs';
import { fetchCustomers } from '@/app/lib/data';

export default async function Page() {
  const customers = await fetchCustomers();

  return (
    <main>
      <Breadcrumbs
        breadcrumbs={[
          { label: 'Invoices', href: '/dashboard/invoices' },
          {
            label: 'Create Invoice',
            href: '/dashboard/invoices/create',
            active: true,
          },
        ]}
      />
      <Form customers={customers} />
    </main>
  );
}
```

Your page is a Server Component that fetches `customers` and passes it to the `<Form>` component. To save time, we've already created the `<Form>` component for you.

Navigate to the `<Form>` component, and you'll see that the form:

- Has one `<select>` (dropdown) element with a list of **customers**.
- Has one `<input>` element for the **amount** with `type="number"`.
- Has two `<input>` elements for the status with `type="radio"`.
- Has one button with `type="submit"`.

On http://localhost:3000/dashboard/invoices/create, you should see the following UI:

![Create invoices page with breadcrumbs and form|960x563](https://nextjs.org/_next/image?url=%2Flearn%2Fdark%2Fcreate-invoice-page.png&w=1920&q=75&dpl=dpl_9VWEnshqwJ4TWYcNesdqAEpT4iWx)

### [2. Create a Server Action](https://nextjs.org/learn/dashboard-app/mutating-data#2-create-a-server-action)

Great, now let's create a Server Action that is going to be called when the form is submitted.

Navigate to your `lib` directory and create a new file named `actions.ts`. At the top of this file, add the React [`use server`](https://react.dev/reference/react/use-server) directive:

/app/lib/actions.ts

```ts
'use server';
```

By adding the `'use server'`, you mark all the exported functions within the file as server functions. These server functions can then be imported into Client and Server components, making them extremely versatile.

You can also write Server Actions directly inside Server Components by adding "use server" inside the action. But for this course, we'll keep them all organized in a separate file.

In your `actions.ts` file, create a new async function that accepts `formData`:

/app/lib/actions.ts

```
'use server'; export async function createInvoice(formData: FormData) {}
```

Then, in your `<Form>` component, import the `createInvoice` from your `actions.ts` file. Add a `action` attribute to the `<form>` element, and call the `createInvoice` action.

/app/ui/invoices/create-form.tsx

```tsx
'use client';
import { customerField } from '@/app/lib/definitions';
import Link from 'next/link';
import {
  CheckIcon,
  ClockIcon,
  CurrencyDollarIcon,
  UserCircleIcon,
} from '@heroicons/react/24/outline';
import { Button } from '@/app/ui/button';
import { createInvoice } from '@/app/lib/actions';

export default function Form({ customers }: { customers: customerField[] }) {
  return <form action={createInvoice}>// ...</form>;
}
```

> **Good to know**: In HTML, you'd pass a URL to the `action` attribute. This URL would be the destination where your form data should be submitted (usually an API endpoint).
>
> However, in React, the `action` attribute is considered a special prop - meaning React builds on top of it to allow actions to be invoked.
>
> Behind the scenes, Server Actions create a `POST` API endpoint. This is why you don't need to create API endpoints manually when using Server Actions.

### [3. Extract the data from `formData`](https://nextjs.org/learn/dashboard-app/mutating-data#3-extract-the-data-from-formdata)

Back in your `actions.ts` file, you'll need to extract the values of `formData`, there are a [couple of methods](https://developer.mozilla.org/en-US/docs/Web/API/FormData/append) you can use. For this example, let's use the [`.get(name)`](https://developer.mozilla.org/en-US/docs/Web/API/FormData/get) method.

/app/lib/actions.ts

```TypeScript
'use server';
export async function createInvoice(formData: FormData) {
  const rawFormData = {
    customerId: formData.get('customerId'),
    amount: formData.get('amount'),
    status: formData.get('status'),
  };
  // Test it out:
  console.log(rawFormData);
}
```

> **Tip:** If you're working with forms that have many fields, you may want to consider using the [`entries()`](https://developer.mozilla.org/en-US/docs/Web/API/FormData/entries) method with JavaScript's [`Object.fromEntries()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/entries). For example:
>
> `const rawFormData = Object.fromEntries(formData.entries())`

To check everything is connected correctly, go ahead and try out the form. After submitting, you should see the data you just entered into the form logged in your terminal.

Now that your data is in the shape of an object, it'll be much easier to work with.

### [4. Validate and prepare the data](https://nextjs.org/learn/dashboard-app/mutating-data#4-validate-and-prepare-the-data)

Before sending the form data to your database, you want to ensure it's in the correct format and with the correct types. If you remember from earlier in the course, your invoices table expects data in the following format:

/app/lib/definitions.ts

```TypeScript
export type Invoice = {
    id: string; // Will be created on the database
    customer_id: string;
    amount: number; // Stored in cents
    status: 'pending' | 'paid';
    date: string;
};
```

So far, you only have the `customer_id`, `amount`, and `status` from the form.

#### [Type validation and coercion](https://nextjs.org/learn/dashboard-app/mutating-data#type-validation-and-coercion)

It's important to validate that the data from your form aligns with the expected types in your database. For instance, if you add a `console.log` inside your action:

```ts
console.log(typeof rawFormData.amount);
```

You'll notice that `amount` is of type `string` and not `number`. This is because `input` elements with `type="number"` actually return a string, not a number!

To handle type validation, you have a few options. While you can manually validate types, using a type validation library can save you time and effort. For your example, we'll use [Zod](https://zod.dev/), a TypeScript-first validation library that can simplify this task for you.

In your `actions.ts` file, import Zod and define a schema that matches the shape of your form object. This schema will validate the `formData` before saving it to a database.

/app/lib/actions.ts

```TypeScript
'use server';
import { z } from 'zod';

const FormSchema = z.object({
    id: z.string(),
    customerId: z.string(),
    amount: z.coerce.number(),
    status: z.enum(['pending', 'paid']),
    date: z.string(),
});

const CreateInvoice = FormSchema.omit({ id: true, date: true });

export async function createInvoice(formData: FormData) {
    // ...
}
```

The `amount` field is specifically set to coerce (change) from a string to a number while also validating its type.

You can then pass your `rawFormData` to `CreateInvoice` to validate the types:

/app/lib/actions.ts

```
// ...export async function createInvoice(formData: FormData) {  const { customerId, amount, status } = CreateInvoice.parse({    customerId: formData.get('customerId'),    amount: formData.get('amount'),    status: formData.get('status'),  });}
```

#### [Storing values in cents](https://nextjs.org/learn/dashboard-app/mutating-data#storing-values-in-cents)

It's usually good practice to store monetary values in cents in your database to eliminate JavaScript floating-point errors and ensure greater accuracy.

Let's convert the amount into cents:

/app/lib/actions.ts

```
// ...export async function createInvoice(formData: FormData) {  const { customerId, amount, status } = CreateInvoice.parse({    customerId: formData.get('customerId'),    amount: formData.get('amount'),    status: formData.get('status'),  });  const amountInCents = amount * 100;}
```

#### [Creating new dates](https://nextjs.org/learn/dashboard-app/mutating-data#creating-new-dates)

Finally, let's create a new date with the format "YYYY-MM-DD" for the invoice's creation date:

/app/lib/actions.ts

```
// ...export async function createInvoice(formData: FormData) {  const { customerId, amount, status } = CreateInvoice.parse({    customerId: formData.get('customerId'),    amount: formData.get('amount'),    status: formData.get('status'),  });  const amountInCents = amount * 100;  const date = new Date().toISOString().split('T')[0];}
```

### [5. Inserting the data into your database](https://nextjs.org/learn/dashboard-app/mutating-data#5-inserting-the-data-into-your-database)

Now that you have all the values you need for your database, you can create an SQL query to insert the new invoice into your database and pass in the variables:

/app/lib/actions.ts

```
import { z } from 'zod';import { sql } from '@vercel/postgres'; // ... export async function createInvoice(formData: FormData) {  const { customerId, amount, status } = CreateInvoice.parse({    customerId: formData.get('customerId'),    amount: formData.get('amount'),    status: formData.get('status'),  });  const amountInCents = amount * 100;  const date = new Date().toISOString().split('T')[0];   await sql`    INSERT INTO invoices (customer_id, amount, status, date)    VALUES (${customerId}, ${amountInCents}, ${status}, ${date})  `;}
```

Right now, we're not handling any errors. We'll do it in the next chapter. For now, let's move on to the next step.

### [6. Revalidate and redirect](https://nextjs.org/learn/dashboard-app/mutating-data#6-revalidate-and-redirect)

Next.js has a [Client-side Router Cache](https://nextjs.org/docs/app/building-your-application/caching#router-cache) that stores the route segments in the user's browser for a time. Along with [prefetching](https://nextjs.org/docs/app/building-your-application/routing/linking-and-navigating#1-prefetching), this cache ensures that users can quickly navigate between routes while reducing the number of requests made to the server.

Since you're updating the data displayed in the invoices route, you want to clear this cache and trigger a new request to the server. You can do this with the [`revalidatePath`](https://nextjs.org/docs/app/api-reference/functions/revalidatePath) function from Next.js:

/app/lib/actions.ts

```
'use server'; import { z } from 'zod';import { sql } from '@vercel/postgres';import { revalidatePath } from 'next/cache'; // ... export async function createInvoice(formData: FormData) {  const { customerId, amount, status } = CreateInvoice.parse({    customerId: formData.get('customerId'),    amount: formData.get('amount'),    status: formData.get('status'),  });  const amountInCents = amount * 100;  const date = new Date().toISOString().split('T')[0];   await sql`    INSERT INTO invoices (customer_id, amount, status, date)    VALUES (${customerId}, ${amountInCents}, ${status}, ${date})  `;   revalidatePath('/dashboard/invoices');}
```

Once the database has been updated, the `/dashboard/invoices` path will be revalidated, and fresh data will be fetched from the server.

At this point, you also want to redirect the user back to the `/dashboard/invoices` page. You can do this with the [`redirect`](https://nextjs.org/docs/app/api-reference/functions/redirect) function from Next.js:

/app/lib/actions.ts

```
'use server'; import { z } from 'zod';import { sql } from '@vercel/postgres';import { revalidatePath } from 'next/cache';import { redirect } from 'next/navigation'; // ... export async function createInvoice(formData: FormData) {  // ...   revalidatePath('/dashboard/invoices');  redirect('/dashboard/invoices');}
```

Congratulations! You've just implemented your first Server Action. Test it out by adding a new invoice, if everything is working correctly:

1. You should be redirected to the `/dashboard/invoices` route on submission.
2. You should see the new invoice at the top of the table.

## [Updating an invoice](https://nextjs.org/learn/dashboard-app/mutating-data#updating-an-invoice)

The updating invoice form is similar to the create an invoice form, except you'll need to pass the invoice `id` to update the record in your database. Let's see how you can get and pass the invoice `id`.

These are the steps you'll take to update an invoice:

1. Create a new dynamic route segment with the invoice `id`.
2. Read the invoice `id` from the page params.
3. Fetch the specific invoice from your database.
4. Pre-populate the form with the invoice data.
5. Update the invoice data in your database.

### [1. Create a Dynamic Route Segment with the invoice `id` ](https://nextjs.org/learn/dashboard-app/mutating-data#1-create-a-dynamic-route-segment-with-the-invoice-id)

Next.js allows you to create [Dynamic Route Segments](https://nextjs.org/docs/app/building-your-application/routing/dynamic-routes) when you don't know the exact segment name and want to create routes based on data. This could be blog post titles, product pages, etc. You can create dynamic route segments by wrapping a folder's name in square brackets. For example, `[id]`, `[post]` or `[slug]`.

In your `/invoices` folder, create a new dynamic route called `[id]`, then a new route called `edit` with a `page.tsx` file. Your file structure should look like this:

![Invoices folder with a nested [id] folder, and an edit folder inside it|1600x444](https://nextjs.org/_next/image?url=%2Flearn%2Fdark%2Fedit-invoice-route.png&w=3840&q=75&dpl=dpl_9VWEnshqwJ4TWYcNesdqAEpT4iWx)

In your `<Table>` component, notice there's a `<UpdateInvoice />` button that receives the invoice's `id` from the table records.

/app/ui/invoices/table.tsx

```
export default async function InvoicesTable({  query,  currentPage,}: {  query: string;  currentPage: number;}) {  return (    // ...    <td className="flex justify-end gap-2 whitespace-nowrap px-6 py-4 text-sm">      <UpdateInvoice id={invoice.id} />      <DeleteInvoice id={invoice.id} />    </td>    // ...  );}
```

Navigate to your `<UpdateInvoice />` component, and update the `href` of the `Link` to accept the `id` prop. You can use template literals to link to a dynamic route segment:

/app/ui/invoices/buttons.tsx

```
import { PencilIcon, PlusIcon, TrashIcon } from '@heroicons/react/24/outline';import Link from 'next/link'; // ... export function UpdateInvoice({ id }: { id: string }) {  return (    <Link      href={`/dashboard/invoices/${id}/edit`}      className="rounded-md border p-2 hover:bg-gray-100"    >      <PencilIcon className="w-5" />    </Link>  );}
```

### [2. Read the invoice `id` from page `params` ](https://nextjs.org/learn/dashboard-app/mutating-data#2-read-the-invoice-id-from-page-params)

Back on your `<Page>` component, paste the following code:

/app/dashboard/invoices/[id]/edit/page.tsx

```
import Form from '@/app/ui/invoices/edit-form';import Breadcrumbs from '@/app/ui/invoices/breadcrumbs';import { fetchCustomers } from '@/app/lib/data'; export default async function Page() {  return (    <main>      <Breadcrumbs        breadcrumbs={[          { label: 'Invoices', href: '/dashboard/invoices' },          {            label: 'Edit Invoice',            href: `/dashboard/invoices/${id}/edit`,            active: true,          },        ]}      />      <Form invoice={invoice} customers={customers} />    </main>  );}
```

Notice how it's similar to your `/create` invoice page, except it imports a different form (from the `edit-form.tsx` file). This form should be **pre-populated** with a `defaultValue` for the customer's name, invoice amount, and status. To pre-populate the form fields, you need to fetch the specific invoice using `id`.

In addition to `searchParams`, page components also accept a prop called `params` which you can use to access the `id`. Update your `<Page>` component to receive the prop:

/app/dashboard/invoices/[id]/edit/page.tsx

```
import Form from '@/app/ui/invoices/edit-form';import Breadcrumbs from '@/app/ui/invoices/breadcrumbs';import { fetchCustomers } from '@/app/lib/data'; export default async function Page({ params }: { params: { id: string } }) {  const id = params.id;  // ...}
```

### [3. Fetch the specific invoice](https://nextjs.org/learn/dashboard-app/mutating-data#3-fetch-the-specific-invoice)

Then:

- Import a new function called `fetchInvoiceById` and pass the `id` as an argument.
- Import `fetchCustomers` to fetch the customer names for the dropdown.

You can use `Promise.all` to fetch both the invoice and customers in parallel:

/dashboard/invoices/[id]/edit/page.tsx

```
import Form from '@/app/ui/invoices/edit-form';import Breadcrumbs from '@/app/ui/invoices/breadcrumbs';import { fetchInvoiceById, fetchCustomers } from '@/app/lib/data'; export default async function Page({ params }: { params: { id: string } }) {  const id = params.id;  const [invoice, customers] = await Promise.all([    fetchInvoiceById(id),    fetchCustomers(),  ]);  // ...}
```

You will see a temporary TS error for the `invoices` prop in your terminal because invoices could be potentially undefined. Don't worry about it for now, you'll resolve it in the next chapter when you add error handling.

Great! Now, test that everything is wired correctly. Visit http://localhost:3000/dashboard/invoices and click on the Pencil icon to edit an invoice. After navigation, you should see a form that is pre-populated with the invoice details:

![Edit invoices page with breadcrumbs and form|960x563](https://nextjs.org/_next/image?url=%2Flearn%2Fdark%2Fedit-invoice-page.png&w=1920&q=75&dpl=dpl_9VWEnshqwJ4TWYcNesdqAEpT4iWx)

The URL should also be updated with an `id` as follows: `http://localhost:3000/dashboard/invoice/uuid/edit`

> **UUIDs vs. Auto-incrementing Keys**
>
> We use UUIDs instead of incrementing keys (e.g., 1, 2, 3, etc.). This makes the URL longer; however, UUIDs eliminate the risk of ID collision, are globally unique, and reduce the risk of enumeration attacks - making them ideal for large databases.
>
> However, if you prefer cleaner URLs, you might prefer to use auto-incrementing keys.

### [4. Pass the `id` to the Server Action](https://nextjs.org/learn/dashboard-app/mutating-data#4-pass-the-id-to-the-server-action)

Lastly, you want to pass the `id` to the Server Action so you can update the right record in your database. You **cannot** pass the `id` as an argument like so:

/app/ui/invoices/edit-form.tsx

```
// Passing an id as argument won't work<form action={updateInvoice(id)}>
```

Instead, you can pass `id` to the Server Action using JS `bind`. This will ensure that any values passed to the Server Action are encoded.

/app/ui/invoices/edit-form.tsx

```
// ...import { updateInvoice } from '@/app/lib/actions'; export default function EditInvoiceForm({  invoice,  customers,}: {  invoice: InvoiceForm;  customers: CustomerField[];}) {  const updateInvoiceWithId = updateInvoice.bind(null, invoice.id);   return (    <form action={updateInvoiceWithId}>      <input type="hidden" name="id" value={invoice.id} />    </form>  );}
```

> **Note:** Using a hidden input field in your form also works (e.g. `<input type="hidden" name="id" value={invoice.id} />`). However, the values will appear as full text in the HTML source, which is not ideal for sensitive data like IDs.

Then, in your `actions.ts` file, create a new action, `updateInvoice`:

/app/lib/actions.ts

```TypeScript
// Use Zod to update the expected types
const UpdateInvoice = FormSchema.omit({ id: true, date: true });

// ...

export async function updateInvoice(id: string, formData: FormData) {
    const { customerId, amount, status } = UpdateInvoice.parse({
        customerId: formData.get('customerId'),
        amount: formData.get('amount'),
        status: formData.get('status'),
    });

    const amountInCents = amount * 100;

    await sql`
        UPDATE invoices
        SET customer_id = ${customerId}, amount = ${amountInCents}, status = ${status}
        WHERE id = ${id}
    `;

    revalidatePath('/dashboard/invoices');
    redirect('/dashboard/invoices');
}
```

Similarly to the `createInvoice` action, here you are:

1. Extracting the data from `formData`.
2. Validating the types with Zod.
3. Converting the amount to cents.
4. Passing the variables to your SQL query.
5. Calling `revalidatePath` to clear the client cache and make a new server request.
6. Calling `redirect` to redirect the user to the invoice's page.

Test it out by editing an invoice. After submitting the form, you should be redirected to the invoices page, and the invoice should be updated.

## [Deleting an invoice](https://nextjs.org/learn/dashboard-app/mutating-data#deleting-an-invoice)

To delete an invoice using a Server Action, wrap the delete button in a `<form>` element and pass the `id` to the Server Action using `bind`:

/app/ui/invoices/buttons.tsx

```
import { deleteInvoice } from '@/app/lib/actions'; // ... export function DeleteInvoice({ id }: { id: string }) {  const deleteInvoiceWithId = deleteInvoice.bind(null, id);   return (    <form action={deleteInvoiceWithId}>      <button className="rounded-md border p-2 hover:bg-gray-100">        <span className="sr-only">Delete</span>        <TrashIcon className="w-4" />      </button>    </form>  );}
```

Inside your `actions.ts` file, create a new action called `deleteInvoice`.

/app/lib/actions.ts

```
export async function deleteInvoice(id: string) {  await sql`DELETE FROM invoices WHERE id = ${id}`;  revalidatePath('/dashboard/invoices');}
```

Since this action is being called in the `/dashboard/invoices` path, you don't need to call `redirect`. Calling `revalidatePath` will trigger a new server request and re-render the table.

## [Further reading](https://nextjs.org/learn/dashboard-app/mutating-data#further-reading)

In this chapter, you learned how to use Server Actions to mutate data. You also learned how to use the `revalidatePath` API to revalidate the Next.js cache and `redirect` to redirect the user to a new page.

You can also read more about [security with Server Actions](https://nextjs.org/blog/security-nextjs-server-components-actions) for additional learning.

12

## You've Completed Chapter 12

Congratulations! You learned how to mutate data using forms and React Server Actions.

Next Up

13: Handling Errors

Let's explore best practices for mutating data with forms, including error handling and accessibility.

[Start Chapter 13](./error-handling.md)
