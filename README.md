# Mie Files for Flarum 2.x

Private file library and upload extension for Flarum 2.x. Files are validated by both normalized extension and PHP `finfo` MIME type, stored outside Flarum's public directory by default, and delivered through an authenticated streaming route.

## Install

```bash
composer require mie/flarum-files:@dev
php flarum extension:enable mie-files
php flarum cache:clear
```

The extension targets Flarum `^2.0@beta` and was verified against `v2.0.0-rc.5`.

## Behavior

- The composer adds `fas fa-file-upload` and `fas fa-photo-video` icon controls.
- Upload supports file selection, drag and drop, multi-file progress, type/size/permission failures, and a per-user file library.
- Double-clicking a library item asks the server to generate the configured insertion template. The browser never constructs an object-storage URL itself.
- Categories use explicit extension/MIME pairs. `ogg` and `webm` may occur in audio and video only when their full detected MIME differs; other duplicate extensions are rejected.
- Built-in presets cover images, PDF, Word, spreadsheets, archives, audio, and video with real extensions and complete MIME strings.
- Category permissions are `mie-files.category.{permission-name}.{view|download|upload}` and appear in Flarum's permission UI as `File category-{permission-name}-{action}`. The extension also registers `mie-files.view-other` and `mie-files.delete-other`.

## Storage

`local` is always available and stores files in `storage/mie-files`, outside the public web root.

DogeCloud uses the AWS S3-compatible SDK. AccessKeyId and AccessKeySecret are encrypted server-side and are never returned by the API. API payloads also omit internal object keys and endpoint values.

- Empty public base URL: Flarum proxy mode. Each delivery is authorized, streamed, and counted; tracked templates honor referrer protection.
- Public base URL set: direct URL mode. The server authorizes generation, then returns the configured public URL using a random object path. PHP statistics, hotlink protection, and per-download authorization cannot apply after that public URL is issued. The admin UI requires explicit confirmation before saving this mode.

## Maintenance

Set the orphan retention period on the plugin settings tab, then run:

```bash
php flarum mie-files:clean-orphans
```

The command is also registered with Flarum's daily scheduler. It only removes successful files older than the retention period with no linked post. Failed object deletion leaves the file row in `delete_failed` for retry rather than claiming success.

## Development checks

```bash
composer validate --strict
composer run lint
composer run analyse
composer run test
composer run test:integration

cd js
npm ci --ignore-scripts
npm run typecheck:common
npm run typecheck:forum
npm run typecheck:admin
npm run test:common
npm run test:forum
npm run test:admin
npm run build
```

## Architecture

The extension is split by runtime boundary rather than feature name. See
[the architecture map](docs/ARCHITECTURE.md) for the request flows, ownership
of each directory, and commands for refreshing the local Graphify code map.
