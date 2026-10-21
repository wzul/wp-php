# wp-php

WordPress-ready PHP images, **built from the official PHP source** so a security
release is installable the same day — without waiting for a base image to be rebuilt.

```bash
docker pull ghcr.io/wzul/wp-php:8.5.12
```

## Why this exists

When PHP ships an out-of-band security release, the official Docker tags and most
third-party WordPress images keep serving the previous build for days:

| what | latest available on 21 Oct 2026 | fixed in |
|---|---|---|
| `php:8.5-fpm-alpine` | 8.5.11 | **8.5.12** |
| `meghsh/php:8.5` | 8.5.8 | **8.5.12** |
| `lsphp84` (LiteSpeed repo) | 8.4.26 | **8.4.27** |

For [PHP2Shell](https://www.php.net/archive/2026.php) (CVE-2026-109618 /
CVE-2026-109619, unauthenticated RCE, CVSS 9.2) that gap is the difference
between patched and exploitable. These images compile the exact tarball from
php.net, so the fix is available as soon as the source is.

## Tags

| tag | PHP | flavour |
|---|---|---|
| `8.5.12`, `8.5` | 8.5.12 | php-fpm (alpine) |
| `8.4.27`, `8.4` | 8.4.27 | php-fpm (alpine) |
| `8.3.36`, `8.3` | 8.3.36 | php-fpm (alpine) |
| `8.2.35`, `8.2` | 8.2.35 | php-fpm (alpine) |

Pin the full version in production; the minor tag moves with patch releases.

## What's inside

PHP extensions — the set WordPress core and the common plugin stack actually use:

| group | modules |
|---|---|
| database | `pdo_mysql` `mysqli` `pdo_pgsql` `pdo_sqlite` `sqlite3` |
| media | `gd` (jpeg/png/webp/freetype) `imagick` `exif` |
| caching | `opcache` `redis` `apcu` `igbinary` `msgpack` |
| core/web | `mbstring` `intl` `zip` `curl` `openssl` `xml` `xsl` `soap` `sockets` `fileinfo` `bcmath` `sodium` |

System binaries the modules shell out to:

- **`ghostscript`** — Imagick's PDF/PS delegate, needed for PDF previews and
  thumbnail generation
- `imagemagick` CLI, `libjpeg-turbo`, `freetype`, `libpng`, `libwebp`, `libzip`,
  `icu-libs`, `libxml2`, `oniguruma`

Anything missing is a one-line addition in the Dockerfile — open an issue with the
extension name and the plugin that needs it.

## Use with WordPress

```yaml
services:
  php:
    image: ghcr.io/wzul/wp-php:8.5.12
    volumes:
      - ./wordpress:/var/www/html
      - ./php.ini:/usr/local/etc/php/conf.d/zz-custom.ini:ro
    restart: always
```

The image keeps the upstream layout: `/usr/local/etc/php/conf.d` for ini files,
FPM listening on `9000`, user `www-data`.

## Verify a pull

```bash
docker run --rm ghcr.io/wzul/wp-php:8.5.12 php -v
docker run --rm ghcr.io/wzul/wp-php:8.5.12 php -m
```

The build fails if `php -v` inside the image does not match the release that the
tarball was downloaded for, so a tag cannot silently point at an older build.

## Building locally

```bash
docker build --build-arg PHP_VERSION=8.5.12 -t wp-php:8.5.12 fpm/
```

## Status

The php-fpm images are used in production on a multi-site WordPress fleet.
A LiteSpeed (`lsphp`) flavour is planned — LiteSpeed's own packages lag releases
the same way, and it can be compiled from the same source with `--enable-litespeed`.
It is not published yet because it has not been validated end to end.

## License

MIT for this build configuration. PHP itself is under the PHP License 3.01.
