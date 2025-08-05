What you’re building (in simple terms)

A web app where people:

    Create a design system (fonts, colors, spacing, radii, shadows).

    Style common UI (buttons, inputs, lists, badges, alerts, toasts).

    Preview & share the system on a public URL (with optional custom domain).

    Export the system as CSS variables + ready-to-use React/Rails snippets.

    Browse a community library, clone systems, and remix.

    Use AI to generate starting points, improve accessibility, and suggest variants.

    Use built-ins: icon library, palette generator, contrast checker.

Recommended stack (swap as you like)

    Frontend: Ruby on Rails + Devise + Tailwind (or plain CSS variables), shadcn/ui (optional), Iconify.

    Auth & DB: Postgres

    Storage: Postgres or filestack for exports & thumbnails.

    Domains/Hosting: Railway

    AI: OpenAI API (responses in JSON for tokens + code blocks).

    Background jobs: Sidekiq for export jobs.

    Design tokens: Store as JSON (W3C Design Tokens-style).

Core data model (tables)

    users: id, email, name, handle, avatar.

    design_systems: id, owner_id, name, slug, visibility (private/unlisted/public), version, published_at.

    token_sets: id, design_system_id, type (color/typography/space/radius/shadow), json (the tokens).

    components: id, design_system_id, kind (button/input/badge/alert/toast/list/…),
    variants (json), code_preview (html/jsx), notes.

    previews: id, design_system_id, public_slug, custom_domain (nullable), og_image_url.

    exports: id, design_system_id, format (css/json/tailwind/react/rails), status, file_url, created_at.

    library_items: id, design_system_id, status (listed/hidden), tags (string[]), downloads_count, forks_count.

    forks: id, source_system_id, target_system_id, user_id.

    audit: id, design_system_id, change_summary, before_json, after_json, created_at.

    Keep json columns typed with a schema validator on write.

Token format (example)

{
"color": {
"primary": { "value": "#3B82F6" },
"primary-600": { "value": "#2563EB" },
"bg": { "value": "#0B0F14" },
"text": { "value": "#E5E7EB" }
},
"typography": {
"fontFamily": { "value": "Inter, system-ui, sans-serif" },
"scale": { "value": { "xs": 12, "sm": 14, "base": 16, "lg": 18, "xl": 20, "2xl": 24 } },
"lineHeight": { "value": 1.6 }
},
"radius": { "sm": { "value": 4 }, "md": { "value": 8 }, "lg": { "value": 12 } },
"space": { "xs": { "value": 4 }, "sm": 8, "md": 12, "lg": 16, "xl": 24 }
}

How it works (flows)

    Create system

        User picks base template or “AI generate”.

        App creates default token sets + default component variants.

        Live preview shows a demo page with their tokens/components applied.

    Edit tokens & components

        Color picker (with palette generator + contrast meter).

        Typography picker (Google Fonts selector, fallback stack).

        Component editors (button states, sizes, corners, shadows, spacing).

        Accessibility panel (WCAG AA checks, suggested fixes).

    Preview & share

        Public URL: /u/{handle}/{slug}.

        Optional custom domain (CNAME to your edge endpoint).

        Social OG image auto-generated from tokens.

    Export

        Generate:

            CSS variables (:root { --color-primary: #... }).

            Tailwind preset (theme extensions).

            React components (Button, Input, Badge) wired to tokens.

            Rails partials (ERB) + SCSS variables.

            Design tokens JSON for other tools.

        Zip and provide a download link.

    Community

        List public systems with filters (color mood, serif/sans, rounded/sharp).

        View details → “Clone to my account”.

        Show attribution and link to original; track forks/downloads.

    AI assistance

        “Describe your vibe” → AI returns tokens + starter variants.

        “Make this AA compliant” → AI nudges tokens.

        “Generate 6 button variants” → AI returns consistent variants + code.

        “Name this palette & components”.

To-Do List for the AI Coder (copy/paste)
Phase 0 — Project setup

Add ViewComponent gem and configure Iconify via importmap; create icon picker component.

Generate controllers and views: DashboardController, EditorController, PreviewController, LibraryController.

Set up Devise for authentication; generate User model with handle field.

Phase 1 — Database & APIs

Generate ActiveRecord models: User, DesignSystem, TokenSet, Component, Preview, Export, LibraryItem, Fork, Audit.

Write Rails migrations with proper foreign keys, indexes, and constraints.

Create API controllers inheriting from ApplicationController:

`rails generate controller Api::DesignSystems` with actions: create, show, update.

`rails generate controller Api::Tokens` with update action.

`rails generate controller Api::Components` with update action.

`rails generate controller Api::Previews` with create action (publish).

`rails generate controller Api::Exports` with create action.

`rails generate controller Api::Library` with index action.

`rails generate controller Api::Forks` with create action.

Add Rails routes in `config/routes.rb` using namespace :api and resources.

Add JSON schema validation using `json_schemer` gem for token and component JSON columns.

Configure Strong Parameters for all controllers.

Phase 2 — Editor UI

Create ViewComponents for token editors: ColorTokenComponent, TypographyTokenComponent, SpaceTokenComponent, RadiusTokenComponent, ShadowTokenComponent.

Build Stimulus controllers for interactive features:

- `color_picker_controller.js` for color selection and palette generation
- `contrast_checker_controller.js` for WCAG compliance validation
- `typography_picker_controller.js` for Google Fonts integration via Web Font Loader
- `live_preview_controller.js` for real-time token updates

Create component editor ViewComponents:

- `ButtonEditorComponent` with variant forms (solid/outline/ghost), size options, state management
- `InputEditorComponent` with size variants, focus ring configuration, validation states
- `BadgeEditorComponent` with color theming and border radius options
- `AlertEditorComponent` with semantic color variants (success/info/warning/error)
- `ToastEditorComponent` with positioning and animation options

Build `PreviewCanvasComponent` with:

- Turbo Streams for real-time updates
- Responsive viewport switcher using CSS media queries
- CSS custom properties injection for live token changes

Phase 3 — Public preview & domains

Add public routes in `config/routes.rb`: `get '/u/:handle/:slug', to: 'public_previews#show'`

Generate `PublicPreviewsController` with show action for read-only system display.

Create OG image generator service using `mini_magick` gem:

- `OgImageGeneratorService` class to create preview images from design tokens
- Store generated images using Active Storage

Implement custom domain support:

- Add `CustomDomainMiddleware` to handle domain routing
- Create `DomainVerificationService` for DNS validation
- Add domain management to Preview model with validation
- Configure subdomain routing and 404 handling

Phase 4 — Exports

Create export service classes:

- `CssExportService` to generate tokens.css with :root variables and components.css
- `TailwindExportService` to output tailwind.preset.js with theme extensions
- `ReactExportService` to generate TypeScript components consuming CSS variables
- `RailsExportService` to output SCSS partials and ViewComponent classes
- `JsonExportService` for raw design tokens

Implement Sidekiq background jobs:

- `ExportJob` to handle async export generation
- Use Active Storage to store generated zip files
- Track export status in Export model (pending/processing/completed/failed)

Add download tracking and file cleanup:

- Increment downloads_count on Export model
- Schedule cleanup job to remove old export files

Phase 5 — Community library

Create `LibraryController` with index and show actions for public system browsing.

Build search and filtering using Ransack gem or pg_search:

- Filter by tags, color mood, typography style
- Search by system name and description
- Pagination using Kaminari gem

Implement fork functionality:

- `ForkService` to duplicate DesignSystem records with associations
- Create Fork model to track source and target relationships
- Add attribution links and fork/download counters

Add moderation features:

- Admin-only hide/report flags on LibraryItem model
- Create AdminController with moderation dashboard
- Use Pundit gem for authorization policies

Phase 6 — AI integration

Generate `Api::AiController` with actions: design_starter, accessibility_fix, components.

Create AI service classes:

- `OpenaiService` for API communication with rate limiting
- `DesignStarterService` to generate initial design tokens from user input
- `AccessibilityFixService` to suggest WCAG-compliant color adjustments
- `ComponentGeneratorService` to create component variants

Add Rails routes in api namespace for AI endpoints.

Implement request/response caching using Rails.cache to reduce API costs.

Add background job `AiRequestJob` for long-running AI operations.

Configure rate limiting using `rack-attack` gem for AI endpoints.

Prompt spec (examples):

    Starter

You are a design-system generator. Output strictly JSON with keys:
color, typography, radius, space, shadow. Ensure WCAG AA for text on bg.
Palette vibe: {adjectives}. Brand: {keywords}. Base font: {family}.

Accessibility

Given tokens JSON and these failing pairs [{fg,bg,ratio}], propose minimal color adjustments to pass WCAG AA. Output the updated tokens JSON only.

Components

        Generate Button, Input, Badge, Alert, Toast variant definitions and example JSX that read CSS variables (e.g., var(--color-primary)). Output JSON field "variants" and "jsx" with code blocks.

Phase 7 — Iconify, palettes, and checks

Create `IconPickerComponent` using Iconify via importmap-rails:

- Stimulus controller for icon search and selection
- Store selected icon names in component variants JSON
- Icon preview in component editors

Build `PaletteGeneratorService`:

- Generate color scales (50/100-900) from seed colors using color theory
- Allow manual adjustments with live preview
- Store palette variations in TokenSet model

Implement `ContrastCheckerService`:

- Calculate WCAG contrast ratios using color luminance
- Label compliance levels (AA/AAA) for text/background pairs
- Provide color adjustment suggestions
- Real-time validation in color picker UI

Phase 8 — Versioning & audit

Implement versioning using PaperTrail gem:

- Track all changes to DesignSystem, TokenSet, and Component models
- Create version history view with JSON diff visualization
- Add "Revert to version" action using PaperTrail's reify method

Enhance Audit model:

- Use Rails callbacks to automatically log significant changes
- Store before/after JSON snapshots for complex changes
- Create audit trail view in admin dashboard

Phase 9 — Polish & DX

Add keyboard shortcuts using Stimulus controllers:

- Cmd/Ctrl+S for saving design systems
- Cmd/Ctrl+P for preview toggle
- Hotkey helpers in ViewComponents

Implement undo/redo functionality:

- Use Stimulus controller with history stack for token edits
- Store edit states in browser sessionStorage
- Keyboard shortcuts for undo (Cmd/Ctrl+Z) and redo (Cmd/Ctrl+Y)

Add clipboard functionality:

- Stimulus controller for copy-to-clipboard actions
- Toast notifications for successful copies
- Copy buttons for generated code snippets

Create JSON import feature:

- File upload form for design token JSON
- Validation and parsing service
- Merge imported tokens with existing system

Phase 10 — Testing & launch

Write RSpec unit tests:

- Model validations and associations
- Service class functionality (token validators, contrast calculations, exporters)
- Controller actions and strong parameters
- ViewComponent rendering

Create system tests using Capybara:

- User authentication flow with Devise
- Complete design system workflow: create → edit → publish → export → clone
- JavaScript interactions using Selenium driver

Add Rails fixtures and FactoryBot factories:

- Seed demo design systems for development and testing
- Create realistic sample data for community gallery

Implement analytics tracking:

- Use Ahoy gem for event tracking
- Track key events: system creation, publishing, exports, clones
- Create analytics dashboard for admin users

Minimal viable product (ship this first)

Set up Devise authentication with User model including handle field.

Create DesignSystem model with base template seeding via Rails fixtures.

Build token editor ViewComponents for colors and typography with live preview using Turbo Streams.

Implement ButtonEditorComponent and InputEditorComponent with basic variant management.

Add PublicPreviewsController with public URL routing (`/u/:handle/:slug`).

Create CssExportService and ReactExportService for basic component generation.

Build OpenaiService integration for "Generate Starter" functionality.

Implement read-only community library with LibraryController and basic ForkService.

You can add Rails-specific exports, additional components (badges/alerts/toasts), custom domain middleware, and advanced AI features in the next sprint.
Notes for Rails implementation details

Theming: Use CSS custom properties with Rails asset pipeline. ViewComponents should read `var(--token-name)` for framework-agnostic styling.

Tailwind integration: Configure `tailwindcss-rails` gem and provide preset generation via TailwindExportService.

Accessibility: Implement ContrastCheckerService with WCAG AA defaults. Use Rails helpers for semantic HTML and ARIA attributes.

Performance: Use Rails caching for token compilation, Turbo for real-time updates, and background jobs for heavy operations.

Security: Validate JSON schemas using `json_schemer` gem, implement rate limiting with `rack-attack`, use Strong Parameters throughout.

Authentication & Authorization: Use Devise for auth, Pundit for authorization policies, and secure API endpoints with proper CSRF protection.

Background Jobs: Use Sidekiq for export generation, AI requests, and file cleanup with proper error handling and retries.

Database: Use PostgreSQL JSON columns with proper indexing, foreign key constraints, and database-level validations.

Licensing: Add license field to LibraryItem model with MIT as default, display license information in community views.
