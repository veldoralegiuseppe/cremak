# AGENTS.md

## Scope and architecture
- This repo is a **LocalWP WordPress instance**: runtime code lives in `app/public`, infra templates in `conf/*`, DB snapshot in `app/sql/local.sql`.
- Treat `app/public/wp-admin` and `app/public/wp-includes` as WordPress core (do not edit directly).
- Treat most plugin/theme code as third-party vendor code; project behavior is mostly configured through DB content.
- Active stack from `app/sql/local.sql` (`wp_options.active_plugins`):
  - `woocommerce/woocommerce.php`
  - `kadence-blocks/kadence-blocks.php`
  - `kadence-starter-templates/kadence-starter-templates.php`
  - `insert-headers-and-footers/ihaf.php` (WPCode)
  - `animations-for-blocks/animations-for-blocks.php`
- Active theme from `wp_options.template` and `wp_options.stylesheet`: `kadence`.

## Where real custom logic currently lives
- Main customizations are stored in DB option `wp_options.wpcode_snippets` inside `app/sql/local.sql` (not in versioned PHP files).
- Existing snippet examples include:
  - JS in `site_wide_footer` for accordion + desktop/mobile menu handling.
  - PHP in `everywhere` and `site_wide_body` hooking `kadence_before_header`, `kadence_main_header`, `wp_footer`.
- These snippets inject custom mega-menu/mobile-menu markup and inline CSS/JS, and rely on Kadence DOM/hooks.

## Editing rules for agents
- Prefer changing project-owned surfaces first:
  1. DB-exported snippet payloads in `app/sql/local.sql` (when the request is about existing menu/header/footer behavior).
  2. New project files under `app/public/wp-content` (for versioned custom code), not core files.
- Do not refactor or patch internals of `woocommerce`, `kadence-*`, or WordPress core unless explicitly requested.
- If behavior ties to a hook, preserve the existing hook contract (`kadence_before_header`, `kadence_main_header`, `wp_footer`).
- Keep copy/labels consistent with current site language (Italian content appears throughout seeded posts/pages).

## Local workflows and debugging
- There is no repo-level build/test pipeline; this is a runtime CMS project.
- LocalWP templated server config is in `conf/nginx/*.hbs`, `conf/php/*.hbs`, `conf/mysql/*.hbs`.
- Runtime logs to inspect first:
  - `logs/php/error.log`
  - `logs/php/php-fpm.log`
  - `logs/nginx/error.log`
- DB snapshot for reproducible state is `app/sql/local.sql`; snippet/custom behavior reviews should include this file.

## Integration notes
- WooCommerce pages and routes are provisioned in DB dump (`shop`, `cart`, `checkout`, `my-account` content in `wp_posts`).
- Kadence + Kadence Blocks provide layout and header hook points; custom snippet code depends on their markup/classes.
- `wp-config.php` indicates local environment (`WP_ENVIRONMENT_TYPE=local`, DB `root/root`, host `localhost`).

## Practical guardrails
- When asked to "change menu/header/footer behavior", inspect DB snippets first, not `themes/kadence/functions.php`.
- When asked to "update plugin/theme", treat updates as vendor operations and isolate any project patch in separate custom code.
- Flag secrets if touched: `app/sql/local.sql` contains license keys and serialized options; avoid exposing them in outputs.
