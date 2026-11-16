
<p align="center">
  <img src="https://raw.githubusercontent.com/storytold/soundcraft/main/assets/app-icon/hicolor/256x256/apps/ai.storyteller.soundcraft.png" width="128" height="128" alt="SoundCraft application icon">
</p>

<h1 align="center">SoundCraft Web — Docker Image</h1>

<p align="center">
  A community-maintained Docker image for
  <a href="https://github.com/storytold/soundcraft">SoundCraft</a>,
  providing its browser-based digital audio workstation through a self-hosted web interface.
  <br>
  Build automation and image publication to GitHub Container Registry (GHCR).
</p>

<p align="center">
  <a href="https://github.com/MikaPST/soundcraft-web/actions/workflows/publish-image.yml">
    <img src="https://github.com/MikaPST/soundcraft-web/actions/workflows/publish-image.yml/badge.svg" alt="Build and publish SoundCraft Web">
  </a>
  <a href="https://github.com/MikaPST/soundcraft-web/releases/latest">
    <img src="https://img.shields.io/github/v/release/MikaPST/soundcraft-web?label=Release&logo=github&color=blue" alt="Latest Release">
  </a>
  <a href="https://github.com/MikaPST/soundcraft-web/pkgs/container/soundcraft-web">
    <img src="https://ghcr-badge.egpl.dev/mikapst/soundcraft-web/latest_tag?trim=major&label=GHCR" alt="GHCR image tag">
  </a>
  <a href="docker-compose.yml">
    <img src="https://img.shields.io/badge/Docker%20Compose-ready-2496ED?logo=docker&logoColor=white" alt="Docker Compose configuration">
  </a>
</p>

---

## Why this repository?

The official SoundCraft project provides a WebAssembly-based version of its digital audio workstation that can be hosted as a static website. This repository builds the web application from the official upstream source code and packages the generated files into a Docker image served by NGINX.

The goals are to:

- Build the SoundCraft Web application from the official upstream source code.
- Publish a ready-to-run Docker image to GHCR.
- Automatically build a versioned image when a new official SoundCraft release is published.
- Publish a matching GitHub Release in this repository, using the official version and release notes.
- Provide a convenient deployment option for Unraid Community Applications.
- Offer a simple Docker Compose configuration for users who prefer managing containers with Compose.
- Keep the packaging and deployment configuration separate from the upstream SoundCraft project.
- Maintain independent, version-controlled Docker and NGINX configurations for consistent and reproducible packaging.
- Support both AMD64 and ARM64 Docker architectures.

This is a packaging and distribution project, not a fork intended to develop a separate digital audio workstation.

## How it works

SoundCraft Web runs directly in the user's browser using WebAssembly. The Docker container serves the compiled static web application through NGINX; it does not provide a remote Linux desktop or perform audio processing on the server.

- **Browser-based DAW:** access SoundCraft's multitrack editing and mixing interface through a modern web browser.
- **WebAssembly application:** the Rust application is compiled to WebAssembly and executes client-side.
- **Audio playback:** playback is handled through the browser's WebAudio API.
- **Graphics acceleration:** the interface uses WebGPU when available and falls back to WebGL2.
- **Demo session:** the web application opens a synthesised demo session by default.
- **Self-hosted:** run the application on your own server and access it over your local network or through a properly configured reverse proxy.
- **No remote desktop:** no VNC, remote desktop session or Linux graphical environment is required.

The container only serves the application files. Audio playback and graphical rendering are handled by the browser rather than the Docker host.

### Important limitations of the web build

The upstream web-hosting documentation specifies that the WebAssembly build does not provide every feature of the native desktop application:

- Audio playback is supported through WebAudio, but the browser may require a click or key press before playback can begin.
- The web build does not support audio recording.
- Native file dialogs are not available.
- The desktop application's control channel is not exposed.
- The application opens the synthesised demo session by default.

The web application supports the following URL parameters:

| Parameter | Behaviour |
|---|---|
| Default | Uses WebGPU when available, otherwise WebGL2 |
| `?webgl` | Forces the WebGL2 rendering backend |
| `?empty` | Starts with an empty session instead of the demo session |

For recording, advanced audio-device configuration and workflows requiring native desktop integration, use the official desktop application.

## Docker image

The image is published to GitHub Container Registry:

```text
ghcr.io/mikapst/soundcraft-web:latest
```

Each official upstream SoundCraft release is also published under its exact version tag, for example:

```text
ghcr.io/mikapst/soundcraft-web:vX.Y.Z
```

Use a versioned tag if you want to keep a deployment on a specific release. The `latest` tag updates when a new image is published.

### Option 1 — Docker Compose (recommended)

A `docker-compose.yml` file is provided at the root of this repository. It is the simplest way to start and maintain SoundCraft Web with Docker Compose.

```yaml
services:
  soundcraft-web:
    image: ghcr.io/mikapst/soundcraft-web:latest
    container_name: soundcraft-web
    ports:
      - "8080:8080"
    restart: unless-stopped
```

From the repository root, start the application with:

```bash
docker compose up -d
```

Open the application in your browser:

```text
http://YOUR_SERVER_IP:8080
```

Replace `YOUR_SERVER_IP` with the IP address or hostname of your server. If Docker is running on your own computer, you can use `http://localhost:8080`.

#### Useful Compose commands

```bash
# View container logs
docker compose logs -f

# Pull the latest image and recreate the container
docker compose pull
docker compose up -d

# Stop and remove the container
docker compose down
```

No `.env` file or persistent volume is required for this basic setup.

The host port is configured in `docker-compose.yml`; by default, port `8080` on the host is mapped to port `8080` in the container.

To use a different host port, change the left-hand side of the mapping. For example, `8081:8080` exposes the application on host port `8081`.

The container serves only the static web application. It does not store audio projects or provide a server-side session repository.

### Option 2 — Run directly with Docker

If you prefer not to use Docker Compose, you can run the container directly:

```bash
docker pull ghcr.io/mikapst/soundcraft-web:latest

docker run -d \
  --name soundcraft-web \
  --restart unless-stopped \
  -p 8080:8080 \
  ghcr.io/mikapst/soundcraft-web:latest
```

Open the application at:

```text
http://YOUR_SERVER_IP:8080
```

Optional CPU and memory limits can be added to the `docker run` command if needed, for example `--cpus=2 --memory=2g`.

Audio playback and graphics rendering run in the browser. Performance and compatibility therefore depend primarily on the client device, browser, audio output and graphics support.

### Network access and reverse proxies

The container listens for HTTP traffic on port `8080`.

For external access, use a reverse proxy such as Traefik to terminate HTTPS and forward requests to the container over HTTP.

HTTPS is required for WebGPU outside `localhost`. Over plain HTTP on other hosts, SoundCraft falls back to WebGL2.

HTTPS also provides the secure context required by certain browser clipboard features.

Do not expose an unencrypted HTTP port directly to the public Internet. Restrict access to your intended network or protect external access with HTTPS and appropriate access controls.

SoundCraft's web build uses WebAudio and does not require `SharedArrayBuffer`, `Cross-Origin-Opener-Policy` or `Cross-Origin-Embedder-Policy` isolation headers.

## Repository structure

This repository maintains its own Docker packaging configuration while retrieving the official SoundCraft application source during automated builds.

```text
soundcraft-web/
├── .github/
│   └── workflows/
│       └── publish-image.yml
├── .dockerignore
├── Dockerfile
├── nginx.conf
├── docker-compose.yml
└── README.md
```

| File | Purpose |
|---|---|
| `Dockerfile` | Defines the NGINX runtime image used to serve the compiled SoundCraft Web application. |
| `nginx.conf` | Configures HTTP delivery, WebAssembly MIME types, compression, caching, security headers and health endpoint. |
| `.dockerignore` | Excludes unnecessary files and directories from the Docker build context. |
| `docker-compose.yml` | Provides a ready-to-use deployment configuration for Docker Compose. |
| `.github/workflows/publish-image.yml` | Automates upstream release detection, Rust and Trunk compilation, multi-platform Docker builds, GHCR publication and GitHub Releases. |
| `README.md` | Documents the project, packaging architecture and deployment methods. |

### Docker build architecture

The Docker image is built using the official SoundCraft source code, combined with the packaging configuration maintained in this repository.

The build process follows these steps:

1. GitHub Actions identifies the latest official SoundCraft release or the version explicitly requested.
2. The corresponding upstream source code is checked out into a temporary `upstream/` directory.
3. The workflow installs the Rust toolchain, the WebAssembly compilation target and Trunk.
4. SoundCraft Web is compiled from `upstream/apps/soundcraft-web` using `trunk build --release`.
5. The generated static application is retrieved from `upstream/dist/web/`.
6. The repository's version-controlled `Dockerfile`, `nginx.conf` and `.dockerignore` are copied into a temporary `docker-context/` directory alongside the compiled application.
7. Docker Buildx uses this prepared context and the repository's Dockerfile to create the NGINX runtime image.
8. The resulting Docker images are published to GHCR for `linux/amd64` and `linux/arm64`.
9. After successful publication, a matching GitHub Release is created or updated.

This approach keeps the application source code aligned with official SoundCraft releases while allowing Docker packaging, runtime configuration and build automation to be maintained independently.

Unlike the previous workflow, the Dockerfile and NGINX configuration are no longer generated dynamically inside GitHub Actions.

Both files are now maintained directly in this repository, making the packaging easier to review, test and update.

The runtime container uses NGINX on port `8080` and provides:

- Static delivery of the compiled SoundCraft Web application.
- Correct MIME types for WebAssembly and JavaScript files.
- Gzip compression for supported assets.
- Appropriate caching for generated application files.
- HTTP security headers.
- A `/healthz` endpoint and Docker health check.
- No requirement for a database, persistent application volume or GPU passthrough.

> [!NOTE]
> The build process follows the currently supported upstream SoundCraft Web structure. Future upstream releases may introduce changes to Rust, Trunk, dependencies or application paths that require corresponding packaging updates.

## Automated builds

GitHub Actions checks the official SoundCraft repository for new published releases.

| Trigger | Behaviour |
|---|---|
| Daily schedule | Checks the latest official SoundCraft release and builds it if it has not already been published here |
| Manual workflow dispatch | Builds the latest official release, or a specific upstream tag when provided |
| Force rebuild | Rebuilds an already published upstream version and updates its MikaPST Release |
| Packaging changes on `main` | Rebuilds the latest official upstream release when the workflow, Dockerfile, NGINX configuration or `.dockerignore` changes |

Each successful build publishes the following GHCR tags:

- `latest` — the most recently packaged official SoundCraft release.
- `<upstream-version>` — the exact upstream release tag, for example `vX.Y.Z`.
- `upstream-<commit>` — the first 12 characters of the upstream source commit used for the image.

The build targets `linux/amd64` and `linux/arm64`.

It checks out the official SoundCraft source at the selected release tag, compiles the WebAssembly application and packages the resulting static files using the version-controlled Dockerfile and NGINX configuration maintained in this repository.

The workflow is idempotent: scheduled and ordinary manual runs skip a version that already has a Release in this repository. A manual run can force a rebuild when needed.

Changes pushed to the monitored packaging files trigger a rebuild of the current latest upstream release.

### GitHub Releases

After a successful Docker build and GHCR push, the workflow creates or updates a [GitHub Release](https://github.com/MikaPST/soundcraft-web/releases) matching the upstream tag. It includes the upstream release notes, source link and commit, Docker tags, supported architectures and (when available) the published image digest.

The Release badge at the top of this README updates automatically. Scheduled runs skip versions already released here. You can force a rebuild from **Actions → Build and publish SoundCraft Web → Run workflow**.

> [!IMPORTANT]
> A GitHub Release is created only after the Docker image has been successfully published. The `latest` tag may change after a rebuild; pin a versioned image tag for a stable deployment.

SoundCraft is under active development. Test new images before relying on them for important work.

## Unraid Community Applications

This image is intended to be distributed through the maintainer's existing Unraid Community Applications repository.

The Unraid template references the published GHCR image and provides configuration for the web port and resource limits.

- **Unraid Community Applications repository:** [MikaPST/unraid-community-apps](https://github.com/MikaPST/unraid-community-apps)
- **SoundCraft Web template:** [soundcraft-web.xml](https://github.com/MikaPST/unraid-community-apps/blob/main/templates/soundcraft-web.xml)

This repository contains the Docker packaging configuration, image build and publication workflow, and Docker Compose configuration.

The Unraid Community Applications template is maintained separately.

## Upstream project

SoundCraft is an open-source digital audio workstation developed by the ArtCraft Team and its contributors.

The application provides multitrack editing and mixing, audio and MIDI tracks, automation, built-in effects and instruments, and audio export capabilities in its native desktop version.

- **Source code:** [storytold/soundcraft](https://github.com/storytold/soundcraft)
- **Official website:** [getartcraft.com/apps/soundcraft](https://getartcraft.com/apps/soundcraft)
- **Official web hosting guide:** [Hosting SoundCraft for the web](https://github.com/storytold/soundcraft/blob/main/packaging/web/README.md)
- **Official releases:** [SoundCraft releases](https://github.com/storytold/soundcraft/releases)

This repository builds the Docker image from the official SoundCraft source at the selected release tag, using independently maintained Docker packaging files.

The MikaPST Release tracks the upstream version but represents community-maintained Docker packaging, not an official SoundCraft release.

## License and attribution

SoundCraft is dual-licensed under the **MIT License or Apache License 2.0**, at the user's option.

Copyright belongs to the ArtCraft Team and SoundCraft contributors.

The upstream project includes license and attribution information for the application and bundled third-party components. These notices and the applicable license files must be preserved when redistributing the application.

- [Upstream MIT License](https://github.com/storytold/soundcraft/blob/main/LICENSE-MIT)
- [Upstream Apache License 2.0](https://github.com/storytold/soundcraft/blob/main/LICENSE-APACHE)
- [Upstream NOTICE](https://github.com/storytold/soundcraft/blob/main/NOTICE)
- [Upstream asset attributions](https://github.com/storytold/soundcraft/blob/main/ATTRIBUTION.md)

The Docker packaging process copies the available upstream license and attribution files into the published runtime image.

The licenses for the upstream application do not automatically determine the license of the packaging files maintained in this repository. Refer to the repository's own `LICENSE` file, if provided, for the terms applicable to those files.

The SoundCraft name, ArtCraft branding and logos remain subject to their respective trademark and usage terms.

This repository is an independent community-maintained packaging project and is **not an official SoundCraft release**. It is not affiliated with or endorsed by the SoundCraft or ArtCraft maintainers.

## Disclaimer

SoundCraft is under active development. Features, compatibility and behaviour may change between builds.

Use the application and its generated audio content at your own discretion.

Consult the official upstream repository for the latest information, limitations and release notes.
