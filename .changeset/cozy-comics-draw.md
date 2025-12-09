---
"@medusajs/admin-vite-plugin": patch
"@medusajs/dashboard": patch
"@medusajs/admin-sdk": patch
---

**What** — What changes are introduced in this PR?

Adds a new `hidden` property to `defineRouteConfig` that accepts a callback function to conditionally hide sidebar
menu items based on the current authenticated user.

**Why** — Why are these changes relevant or necessary?

Currently, there's no built-in way to conditionally show/hide custom admin routes in the sidebar based on user
permissions or roles. Developers need this capability to:

- Implement permission-based menu visibility (e.g., admin-only routes)
- Create role-based feature access in the admin dashboard
- Hide sensitive or advanced features from certain user groups

**How** — How have these changes been implemented?

1. **Type Definition** (`admin-sdk`): Added `hidden` callback to `RouteConfig` interface
2. **Vite Plugin** (`admin-vite-plugin`): Extended AST parsing to detect and pass through the `hidden` property
   reference
3. **Dashboard Types**: Added `hidden` to `MenuItemExtension` and `INavItem` types
4. **Layout Components**: Updated `CoreRouteSection`, `ExtensionRouteSection`, and `SettingsSidebar` to filter out
   items where `hidden(user)` returns `true`
5. **Documentation**: Added "Conditionally Hide Sidebar Item" section to the UI Routes documentation

The implementation follows the same pattern as the existing `icon` property - detecting presence via AST and
passing a reference to the actual config object.

**Testing** — How have these changes been tested, or how can the reviewer test the feature?

- Added unit tests in `generate-menu-items.spec.ts` to verify the `hidden` property is correctly parsed and
  included in generated menu items
- Updated all existing test expectations to include `hidden: undefined` for backwards compatibility verification
- Manual testing: Create a route with `hidden: (user) => !user?.metadata?.roles?.includes("admin")` and verify it
  appears/disappears based on user metadata

---

## Examples

```tsx
// src/admin/routes/admin-only/page.tsx
import { defineRouteConfig } from "@medusajs/admin-sdk"
import { LockClosedSolid } from "@medusajs/icons"
import { Container, Heading } from "@medusajs/ui"

const AdminOnlyPage = () => {
  return (
    <Container className="divide-y p-0">
      <div className="flex items-center justify-between px-6 py-4">
        <Heading level="h2">Admin Only Content</Heading>
      </div>
    </Container>
  )
}

export const config = defineRouteConfig({
  label: "Admin Only",
  icon: LockClosedSolid,
  // Hide from users without "admin" role in their metadata
  hidden: (user) => !user?.metadata?.roles?.includes("admin"),
})

export default AdminOnlyPage

```

The callback receives the current authenticated user object (or null) with properties: id, email, first_name,
last_name, and metadata. Return true to hide the menu item, false to show it.

---
Additional Context

Important Security Note: The hidden property only hides the sidebar menu item - it does NOT protect the route
itself. Users can still access hidden routes via direct URL navigation. For true route protection, server-side
middleware or API route guards should be implemented.

Backwards Compatibility: The hidden property is optional, so all existing routes continue to work unchanged.

--- 

Files Changed:
- packages/admin/admin-sdk/src/config/types.ts
- packages/admin/admin-vite-plugin/src/routes/generate-menu-items.ts
- packages/admin/admin-vite-plugin/src/routes/__tests__/generate-menu-items.spec.ts
- packages/admin/dashboard/src/dashboard-app/types.ts
- packages/admin/dashboard/src/dashboard-app/dashboard-app.tsx
- packages/admin/dashboard/src/components/layout/nav-item/nav-item.tsx
- packages/admin/dashboard/src/components/layout/main-layout/main-layout.tsx
- packages/admin/dashboard/src/components/layout/settings-layout/settings-layout.tsx
- www/apps/book/app/learn/fundamentals/admin/ui-routes/page.mdx
