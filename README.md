# Stackline Full Stack Assignment

## Overview

This is a sample eCommerce website that includes:
- Product List Page
- Search Results Page
- Product Detail Page

The application contains various bugs including UX issues, design problems, functionality bugs, and potential security vulnerabilities.

## Getting Started

```bash
yarn install
yarn dev
```

## Your Task

1. **Identify and fix bugs** - Review the application thoroughly and fix any issues you find
2. **Document your work** - Create a comprehensive README that includes:
   - What bugs/issues you identified
   - How you fixed each issue
   - Why you chose your approach
   - Any improvements or enhancements you made

We recommend spending no more than 2 hours on this assignment. We are more interested in the quality of your work and your communication than the amount of time you spend or how many bugs you fix!

## Submission

- Fork this repository
- Make your fixes and improvements
- **Replace this README** with your own that clearly documents all changes and your reasoning
- Provide your Stackline contact with a link to a git repository where you have committed your changes

We're looking for clear communication about your problem-solving process as much as the technical fixes themselves.


## Bug #1 – Invalid `src` prop on Next/Image

**Issue:**  
When selecting products under the “Virtual Currency” category, the app crashed with the following error:  
`Invalid src prop on next/image, hostname "images-na.ssl-images-amazon.com" is not configured`.

**Root Cause:**  
Next.js requires all external image domains to be explicitly whitelisted in the `next.config.ts` file.

**Fix:**  
Added the following configuration to allow images from both Amazon domains:
```ts
images: {
  remotePatterns: [
    { protocol: 'https', hostname: 'images-na.ssl-images-amazon.com', pathname: '/**' },
    { protocol: 'https', hostname: 'm.media-amazon.com', pathname: '/**' },
  ],
}
```
**Result:**  
Product images from both domains now load correctly without breaking the application.

---

## Bug #2 – TypeError: `undefined is not an object (evaluating 'product.imageUrls[0]')`

**Issue:**  
When selecting certain categories like “Desktops,” the app crashed because some products didn’t include an `imageUrls` array.

**Root Cause:**  
The code accessed `product.imageUrls[0]` directly without checking if the array existed.

**Fix:**  
Added optional chaining and a user-friendly fallback message for missing images:
```tsx
{product?.imageUrls?.[0] ? (
  <Image src={product.imageUrls[0]} alt={product.title || "Product image"} />
) : (
  <div>No image available</div>
)}
```
**Result:**  
Products that don’t have images now render properly with a placeholder message instead of crashing the app.

---

## Bug #3 – Search Only Worked Within Current Category

**Issue:**  
When performing a search while a category was selected, results only showed products from that specific category instead of searching across all products.

**Root Cause:**  
The frontend always sent both `search` and `category` parameters to `/api/products`, which limited search results to the selected category. The backend also defaulted to returning only 20 results, further limiting search coverage.

**Fix:**  
Updated both the frontend and backend to enable global search across all products. In the frontend (`app/page.tsx`), modified the fetch logic to ignore category filters when a search query is present and increased the limit for broader searches:  
```tsx
if (search) {
  params.append("search", search);
  params.append("limit", "1000");
} else {
  if (selectedCategory) params.append("category", selectedCategory);
  if (selectedSubCategory) params.append("subCategory", selectedSubCategory);
  params.append("limit", "20");
}
```
Also added logic to automatically clear filters when a user begins typing:  
```tsx
onChange={(e) => {
  const value = e.target.value;
  setSearch(value);
  if (value) {
    setSelectedCategory(undefined);
    setSelectedSubCategory(undefined);
  }
}}
```
In the backend (`/api/products/route.ts`), increased the result limit to ensure full dataset search coverage:  
```ts
limit: searchParams.get('limit') ? parseInt(searchParams.get('limit')!) : 1000,
```
**Result:**  
Search now functions globally across all products, regardless of category filters, improving discoverability and overall user experience.

---

## Bug #4 – Subcategory Dropdown Displayed All Options

**Issue:**  
When selecting a product category (e.g., “Cameras”), the subcategory dropdown incorrectly displayed all subcategories across every category, instead of showing only those related to the selected category.

**Root Cause:**  
The frontend `useEffect` fetching subcategories called `/api/subcategories` without passing any category filter, causing the API to return all subcategories. Additionally, the backend route for `/api/subcategories` did not handle category-based filtering.

**Fix:**  
Updated both the frontend and backend to filter subcategories correctly based on the selected category. In the frontend (`app/page.tsx`), the fetch call inside the subcategory `useEffect` was updated to include the selected category as a query parameter:  
```ts
fetch(`/api/subcategories?category=${encodeURIComponent(selectedCategory)}`)
```  
This ensures only relevant subcategories are retrieved when a category is selected. In the backend (`app/api/subcategories/route.ts`), logic was added to filter subcategories by the given category:  
```ts
export async function GET(req: Request) {
  const { searchParams } = new URL(req.url);
  const category = searchParams.get("category");

  const filteredSubCategories = category
    ? allSubCategories.filter((sub) => sub.parentCategory === category)
    : allSubCategories;

  return Response.json({ subCategories: filteredSubCategories });
}
```  
**Result:**  
The subcategory dropdown now dynamically updates to display only the subcategories that belong to the selected category (e.g., selecting “Cameras” only shows “Instant Cameras” and “DSLR Cameras”).

---

## Enhancement – Added Buttons for Easier Navigation

**Issue:**  
Users had to manually refresh to return to the home page after viewing a product or selecting a category.

**Fix:**  
Added a back button on the product detail page:  
```tsx
<Button variant="outline" onClick={() => router.push("/")}>
  ← Back to Products
</Button>
```
Also added this under `app/page.tsx` to include a “Back to Home Page” button when filters are active:  
```tsx
{(selectedCategory || selectedSubCategory) && (
  <div className="mb-4">
    <Button
      variant="outline"
      onClick={() => {
        setSelectedCategory(undefined);
        setSelectedSubCategory(undefined);
      }}
    >
      ← Back to Home Page
    </Button>
  </div>
)}
```
**Why:**  
These additions improve navigation and user experience by allowing users to easily return to the main product list or clear filters without refreshing the page.



