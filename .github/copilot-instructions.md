This repository is a WordPress plugin (OpenBook) that fetches book metadata from Open Library and renders it using user-editable templates.

Quick context

- Plugin entry: `openbook.php` (registers hooks, shortcodes, and the TinyMCE button).
- Admin UI: `openbook_options.php` provides the settings page and saves plugin options.
- Libraries: `libraries/` contains helpers split by concern:
  - `openbook_constants.php` — option names & default template values
  - `openbook_language.php` — user-facing strings / translations
  - `openbook_openlibrary.php` — Open Library API parsing and book-model helpers
  - `openbook_html.php` — HTML building for OB\_ prefixed display elements (used to render templates)
  - `openbook_utilities.php` — network (cURL), option defaults, and small utilities
  - `openbook_button.js` — TinyMCE editor button client code

Big-picture architecture (how things flow)

- The shortcode handler `openbook_insertbookdata` in `openbook.php` is the main runtime entry for rendering. It:
  1. Builds an `openbook_arguments` object from shortcode attributes / saved options.
  2. Uses `openbook_openlibrary_bookdata` to call Open Library's Books API and retrieve JSON.
  3. Extracts fields via helpers in `openbook_openlibrary.php` and formats them via `openbook_html.php`.
  4. Substitutes placeholders (e.g., `[OB_TITLE]`, `[OL_AUTHORFIRST]`) in the chosen template and returns HTML.

Project-specific conventions and patterns

- Options and language constants: almost every configurable label or default lives in `openbook_constants.php` and `openbook_language.php`. When adding an option, update both files and call `openbook_utilities_setDefaultOptions()` as needed.
- Template tokens: templates use bracketed tokens like `[OB_COVER_MEDIUM]` and `[OL_TITLE]`. Token generation lives in `openbook_html.php`; token names are used across the codebase (search for `OB_` and `OL_`).
- Shortcode formats: supports new-style attributes (e.g., `[openbook booknumber="ISBN:1234567890" templatenumber="2"]`) and a legacy content-based format (see `openbook_arguments` parsing in `openbook.php`). Prefer new-style shortcodes for changes.
- Error handling: functions throw Exceptions with language constants; the shortcode wrapper catches and converts them to a human message via `openbook_getDisplayMessage()`.

Integration points & external dependencies

- Open Library Books API: built URL in `openbook_openlibrary.php` (domain from option `openbook_openlibrary_domain`). Network calls use cURL via `openbook_utilities_getUrlContents()` — cURL must be enabled and the plugin checks this on activation.
- TinyMCE: `openbook_button.js` registers an editor button and uses an AJAX action `openbook_action_callback` (see `add_action('wp_ajax_openbook_action', ...)` in `openbook.php`). Keep JS and AJAX parameter names in sync with server-side POST keys: `booknumber`, `templatenumber`, `publisherurl`, `revisionnumber`.

Developer workflows & commands

- There are no build tools; this is plain PHP/JS/CSS. Typical dev workflow is:
  - Edit plugin files under this folder.
  - Drop the plugin into a WordPress installation (or use a local WP dev environment) and activate it.
  - Use the Options page (Settings → OpenBook) to set `Library Domain` (default: `http://openlibrary.org`) and templates.
  - Test shortcodes in posts/pages or use the TinyMCE button.

Files to inspect when changing behavior

- `openbook.php` — main control flow, shortcode parsing, activation hooks, AJAX callback.
- `openbook_openlibrary.php` — any API or JSON parsing changes.
- `openbook_html.php` — HTML rendering, token creation, and safety/escaping rules.
- `openbook_utilities.php` — network behavior (timeouts, proxies), option initialization.
- `openbook_options.php` — settings saving/validation and the admin UI.

Examples (concrete snippets you can reuse)

- API URL construction (Open Library Books API): see `openbook_openlibrary.php` constructing
  `{$domain}/api/books?bibkeys={$bibkeys}&jscmd=data&format=json`.
- Token replacement: `openbook_insertbookdata` uses `str_ireplace('[OB_TITLE]', $OB_TITLE, $display)` to substitute tokens.

Testing and debugging tips

- Enable detailed errors in Settings → OpenBook (Show Error Details) to surface cURL/JSON errors.
- If network calls fail, check `openbook_utilities_getUrlContents()` (it throws specific exceptions for timeout and cURL error). Reproduce API calls with curl in a shell using the same URL.
- For template output, edit Template 1 in Settings and place `[openbook booknumber="ISBN:..." ]` into a post to see final HTML.

Do not change

- Option keys defined in `openbook_constants.php` — renaming them will silently break stored settings.
- The names of shortcode tokens (OB*\*, OL*\*) without updating all replacements in `openbook_insertbookdata`.

If you add features

- Add language strings to `openbook_language.php`; add default values to `openbook_constants.php`; update `openbook_utilities_setDefaultOptions()` to add/remove option defaults; expose new settings in `openbook_options.php`.

If anything in this file is unclear or you'd like more examples (e.g., typical template edits or how tokens map to Open Library fields), tell me which area to expand.
