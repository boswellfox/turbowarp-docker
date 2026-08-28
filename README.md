# turbowarp-docker

Container images for self-hosting [TurboWarp](https://turbowarp.org) components. Each component is a git submodule pointing at the upstream TurboWarp source, wrapped in its own multi-stage Docker build.

## Components

| Directory       | Submodule (upstream)                                                  | Image behavior                                                                     |
| --------------- | --------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| `gui/`          | [`TurboWarp/scratch-gui`](https://github.com/TurboWarp/scratch-gui)   | Builds the TurboWarp-edited Scratch editor and serves it as static files via nginx |
| `packager/`     | [`turbowarp/packager`](https://github.com/turbowarp/packager)         | Builds the TurboWarp packager and serves it as static files via nginx              |
| `cloud-server/` | [`turbowarp/cloud-server`](https://github.com/turbowarp/cloud-server) | Runs the TurboWarp cloud server as a Node.js service on port `9080`                |
| `extensions/`   | [`turbowarp/extensions`](https://github.com/turbowarp/extensions)     | Builds the TurboWarp extensions website and serves it as static files via nginx    |

## Architecture

The `gui`, `packager`, and `extensions` images use a common pattern:

1. A multi-stage build stage (`node:20`/`node:22`) installs dependencies with `npm ci` and runs the project's production build.
2. The build output is copied into an `nginx:alpine` runtime image.

For these images the static frontend would normally need a running cloud server to function (e.g. for cloud variables / project data).

`cloud-server` differs: it runs directly on `node:20` and serves the API via an Express-style server listening on port `9080`, rather than static nginx files.

Reusable static-asset caching rules are defined in the shared `nginx.conf` included in the frontend images.

## Building an image

Submodules must be initialized and updated first:

```sh
git submodule update --init --recursive
```

Then build any component and run it:

```sh
docker build -t turbowarp-gui ./gui
docker run -p 8080:80 turbowarp-gui
```

For the cloud server:

```sh
docker build -t turbowarp-cloud-server ./cloud-server
docker run -p 9080:9080 turbowarp-cloud-server
```

## Continuous integration

- **CI** (`.github/workflows/ci.yml`) runs on PRs and lints every `Dockerfile` with hadolint and validates every `nginx.conf` syntax, discovering files dynamically.
- **CD** (`.github/workflows/cd.yml`) runs on tags matching `*-v*.*.*`. The tag is parsed into a service name and version, then the component is built and pushed to GHCR as `ghcr.io/<owner>/<repo>/<service>:<version>` plus a `:latest` tag.
- **Dependabot** (`.github/dependabot.yml`) checks daily for submodule updates.

## License

turbowarp-docker is under the [MIT](./LICENSE) license.
