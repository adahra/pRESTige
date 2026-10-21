# pRESTige — agent instructions

Plain PHP low-code platform. No package manager, no build step, no test runner, no linter, no CI. Verification is by running the PHP built-in server and opening URLs in a browser.

## Run it
```
php -S 0.0.0.0:8080
# or
./serve.sh
```
Dashboard: http://localhost:8080/

Requires PHP 5.4+ and a MySQL server. Suggested extensions (from README): php-cli, php-common, php-mbstring, php-gd, php-intl, php-xml, php-mysql, php-zip, php-curl, php-mcrypt.

## First-time setup (important)
On first run `api/config.php` does not exist, so the app falls through to the browser setup wizard at `api/configure/`, which writes the gitignored files `api/prestige.config` and `api/prestige.key` (encrypted DB config). Do not commit those; `.gitignore` already excludes them plus `api/uploads` and `ide/{workspace,data,config.php}`.

If you prefer to configure by hand, copy `api/config.sample.php` to `api/config.php` and set `DBHOST`/`DBNAME`/`DBUSER`/`DBPASSWORD`.

## Default credentials
`admin` / `admin` for API Configuration, Code Editor (IDE), and Terminal. Change after first login.

## Component entry points (all under http://localhost:8080/)
| Component | Path | Notes |
|---|---|---|
| API engine | `api/` | `api/index.php` is the entrypoint; MySQL tables become REST APIs automatically |
| API docs | `api/docs/` | Swagger docs + test tool |
| API tester | `api/test/` | Browser-based REST test UI (static files, not unit tests) |
| DB admin | `db/` | Adminer (`db/index.php`) |
| Code editor (new) | `editor/` | `editor/index.html`, `editor/server.php` |
| IDE (Codiad) | `ide/` | `ide/index.php`; custom PHP APIs go in `ide/workspace/api` |
| Builder | `builder/` | `builder/index.html` |
| Node / Python / Lua managers | `node/`, `python/`, `lua/` | run native runtimes under PHP-only hosting |
| File manager | `fm/` | `fm/index.php` |
| Terminal | `terminal/` | web-based shell (does not work on shared hosting) |
| Launch dashboard | `launch/` | `launch/index.html` |

## Writing code
- Custom APIs: create `.php` files inside the `api` project in the Code Editor (`ide/workspace/api`). Every PHP file there is auto-picked up by the API engine; code outside functions runs as a request interceptor. Do not put non-pRESTige PHP in the `api` project (causes 500 errors).
- Event hooks: `before_<method>_<route>($data)` / `on_<method>_<route>($result)` defined in the api project.
- Core engine object is global `$prestige` (alias of `ResterController`) — methods: `register`, `query`, `find`, `findOne`, `create`, `update`, `delete`, `showResult`, `showError`, `getURL`, `postURL`, `sendMail`, `encrypt`/`decrypt` (use `generateCryptoKey()`), etc.
- API version is driven by `API_VERSION` define in `api/config.php`, which must match a file in `api/versions/`.

## Verification
There are no unit tests, no lint, no typecheck, and no CI workflows in this repo. The only verification is starting the server (`php -S 0.0.0.0:8080`) and hitting the URLs above. `api/test/` is the REST Test Test UI, not a test suite — do not treat it as one.

## Vendored / submodule notes
- `api/vendor/` is committed (PHPMailer, Guzzle, OAuth2 libs, defuse/crypto PHAR) — autoloaded via `api/vendor/autoload.php` in `api/index.php`.
- `.gitmodules` references `doc/swagger-php` and `doc/swagger-ui`; those paths are not present in the working tree (submodules not initialized). Do not rely on them.
- `ide/` is vendored Codiad with its own `composer.json`, `build.xml`, `.travis.yml` — treat as third-party, do not modify unless working on the editor itself.
