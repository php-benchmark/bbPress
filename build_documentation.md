# Build / Run Documentation

## Project summary

- Project name: bbPress
- Language: PHP
- Framework: WordPress plugin (bbPress 2.7.0-alpha-2)
- Package manager: Composer (dev-only dependencies)
- Runtime entrypoint: `src/bbpress.php` (WordPress loads this when the plugin is activated)

## Detected project files

- Dependency files: `composer.json`, `package.json`, `package-lock.json`
- Framework files: `src/bbpress.php` (plugin bootstrap), `src/includes/` (component code), `src/templates/default/` (default theme templates)
- Docker files: none
- CI/config files: `phpcs.xml.dist`, `phpunit.xml.dist`, `Gruntfile.js`
- Existing documentation reviewed: `src/readme.txt`, `composer.json` scripts

## How to install dependencies

Commands:

```bash
composer install
npm install
```

Notes:

- `composer.json` declares dev dependencies (PHPCS, PHPUnit, polyfills) plus three benchmark runtime sinks: `mongodb/mongodb`, `twig/twig`, and `lcobucci/jwt` (`^3.4`, used by the CWE-347 canonical sink).
- The `vendor/` directory was not present at baseline; running `composer install` is required to use the PHPCS/PHPUnit binaries and to install `lcobucci/jwt` so the CWE-347 sink resolves at runtime.
- `vendor/lcobucci/jwt` is **not** installed on this host yet — run `composer install` to pull it before runtime-exercising the CWE-347 flow. The `^3.4` constraint matches the v3.x `\Lcobucci\JWT\Parser()->parse()` / `->getClaim()` API used in the sink.

## How to validate syntax/build

Commands:

```bash
# Per-file syntax check (preferred; works without WordPress runtime)
php -l <modified-file>

# Composer manifest sanity (if Composer is installed)
composer validate

# Coding-standards lint (if vendor/ is populated)
composer lint
```

Expected result:

`php -l` should print `No syntax errors detected in <file>` for every modified PHP file.

## How to run the project locally

Commands:

```bash
# Inside a WordPress install:
# 1. Copy the bbPress plugin into wp-content/plugins/bbpress/ (so the plugin loader sees src/bbpress.php).
# 2. Activate via WP Admin > Plugins, or via WP-CLI:
wp plugin activate bbpress
```

URL or entrypoint, if applicable:

- WordPress front-end forum URLs registered by bbPress at runtime (e.g., `/forums/`).
- WP Admin tools page (`wp-admin/tools.php?page=bbp-repair`, `?page=bbp-upgrade`, `?page=bbp-converter`, `?page=bbp-reset`).
- WP Admin settings (`wp-admin/options-general.php?page=bbpress`).

## How to run tests

Commands:

```bash
composer test
# or
./vendor/bin/phpunit -c phpunit.xml.dist
```

Notes:

- The PHPUnit suite under `tests/` targets a WordPress test scaffold. It cannot run without a configured WordPress test environment and a working PHP CLI. No baseline run was performed in this benchmark session.

## Baseline verification before vulnerability planting

Command attempted:

```bash
php -l <every modified file>
```

Result:

NOT EXECUTED — no PHP CLI is available on this benchmarking host (`php -v` returns "command not found" on both bash and PowerShell). `vendor/` is also empty, so `composer lint` and `phpunit` cannot be invoked here.

Notes:

- The lack of a PHP CLI is a property of the benchmarking host, not a pre-existing defect in bbPress. The sink-update pass for CWE-347/918/400 was applied on branch `main` (the working tree carries the uncommitted updates over commit `24f1baf`).
- Manual review (re-reading each modified file end-to-end after every edit) is used to confirm the syntactic correctness of planted code.
- If `php -l` becomes available, run it against each file listed in the final report.



## Verification after modifications

Commands to run after each CWE is planted/updated (when a PHP CLI is available):

```bash
# Updated-sink files (this pass)
php -l src/includes/common/functions.php   # CWE-918 sink
php -l src/includes/forums/functions.php   # CWE-400 sink
php -l src/includes/users/functions.php    # CWE-347 sink
php -l src/includes/topics/functions.php   # CWE-400 source / CWE-918 reachability
composer validate                          # confirms lcobucci/jwt require entry
composer install                           # installs lcobucci/jwt for the CWE-347 sink

# Other previously-planted files
php -l src/includes/admin/tools.php
php -l src/includes/admin/common.php
php -l src/includes/users/signups.php
php -l src/includes/admin/settings.php
php -l src/includes/admin/forums.php
php -l src/includes/extend/akismet.php
```

In the absence of a PHP CLI, perform a full re-read of each modified file and inspect for unbalanced braces, missing semicolons, missing `defined( 'ABSPATH' )` guards, and stray markers.

## PHP-specific notes

PHP does not compile like Rust, Scala, Java, or Go. For this project the meaningful checks are:

- `php -l` for syntax (requires PHP CLI).
- `composer validate` for `composer.json` correctness.
- `phpunit` for the test suite (requires the WordPress test scaffold).
- WordPress runtime activation to confirm the plugin loads without fatal errors (requires a WordPress install).

## Known limitations

- No PHP CLI on the benchmarking host — automated `php -l` was skipped.
- No `vendor/` directory at baseline — PHPCS/PHPUnit binaries unavailable.
- No WordPress installation here — runtime smoke-testing of the plugin is out of scope.
- Verification of the planted code therefore relies on careful post-edit re-reading of each file plus the verification checklist in the skill.
