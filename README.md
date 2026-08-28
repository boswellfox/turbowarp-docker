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

1. A multi-stage build stage (`node:22`) installs dependencies with `npm ci` and runs the project's production build.
2. The build output is copied into an `nginx:alpine` runtime image.

The cloud server is optional: it provides cloud-variable hosting when the frontend is configured to use it, but the static sites do not require a running cloud server to function.

`cloud-server` differs: it runs directly on `node:22` and serves the API via an Express-style server listening on port `9080`, rather than static nginx files.

Each frontend image carries its own component-specific nginx configuration, such as the `nginx.conf` copied in by `gui/Dockerfile`, rather than a single shared `nginx.conf`.

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
docker run -p 127.0.0.1:9080:9080 turbowarp-cloud-server
```

The cloud server does not terminate TLS itself; in production, expose it through a reverse proxy that terminates TLS for the frontend.

## Continuous integration

- **CI** (`.github/workflows/ci.yml`) runs on PRs and lints every `Dockerfile` with hadolint and validates every `nginx.conf` syntax, discovering files dynamically.
- **CD** (`.github/workflows/cd.yml`) runs on tags matching `*-v*.*.*`. The tag is parsed into a service name and version, then the component is built and pushed to GHCR as `ghcr.io/<owner>/<repo>/<service>:<version>` plus a `:latest` tag.
- **Dependabot** (`.github/dependabot.yml`) checks daily for submodule updates.

## License

turbowarp-docker is under the [MIT](./LICENSE) license.
