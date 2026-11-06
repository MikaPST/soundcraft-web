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

The official SoundCraft project provides a WebAssembly-based version that can be hosted as a static website. This repository packages the web application into a Docker image to simplify self-hosting.

The goals are to:

- Publish a matching GitHub Release only after the Docker build and push succeed.
- Build the SoundCraft Web application from the upstream source code.
- Publish a ready-to-run Docker image to GHCR.
- Automatically rebuild the image every week to incorporate upstream changes.
- Provide a convenient deployment option for Unraid Community Applications.
- Offer a simple Docker Compose configuration for users who prefer managing containers with Compose.
- Keep the packaging and deployment configuration separate from the upstream SoundCraft project.

This is a packaging and distribution project, not a fork intended to develop a separate digital audio workstation.

## How it works

SoundCraft Web runs directly in the user's browser using WebAssembly. The Docker container serves the static web application through NGINX; it does not provide a remote Linux desktop or a server-side audio-processing service.

- **Browser-based DAW:** use SoundCraft's multitrack editing and mixing interface in a modern web browser.
- **WebAssembly application:** the application is compiled to WebAssembly and runs client-side.
- **Audio playback:** playback is routed through the browser's WebAudio API.
- **Web graphics:** the interface uses WebGPU when available and automatically falls back to WebGL2.
- **Demo session:** the web build opens the synthesised demo session.
- **Self-hosted:** serve the static application from your own server and access it over your local network or through a properly configured reverse proxy.

The container only serves the static application files. SoundCraft's web build has no server-side code.

### Important limitations of the web build

The upstream web-hosting documentation specifies that the WebAssembly build is not equivalent to the native desktop application:

- Audio playback is supported through WebAudio, but the browser may require a click or key press before audio playback can begin.
- The web build does not support audio recording.
- Native file dialogs are not available in the web build.
- The web build does not expose the desktop application's control channel.
- The web build opens the synthesised demo session rather than providing the full native desktop session workflow.

For recording, audio-device configuration and workflows requiring native desktop integration, use one of the official desktop builds instead.

## Docker image

The image is published to GitHub Container Registry:

```text
ghcr.io/mikapst/soundcraft-web:latest
```

Each official upstream release is also published under its exact version tag, for example:

```text
ghcr.io/mikapst/soundcraft-web:vX.Y.Z
```

Use a versioned tag if you want to keep a deployment on a specific release. The `latest` tag updates when an image is published.

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

No `.env` file or persistent volume is required for this basic setup. The host port is configured in `docker-compose.yml`; by default, port `8080` on the host is mapped to port `8080` in the container. To use a different host port, change the left-hand side of the mapping. For example, `8081:8080` exposes the application on host port `8081`.

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

Audio playback and graphics rendering run in the browser. Performance and compatibility therefore depend on the client device, browser, audio output and graphics support, not only on the Docker host.

### Network access and reverse proxies

The container listens for HTTP traffic on port `8080`. For external access, use a reverse proxy such as Traefik to terminate HTTPS and forward requests to the container over HTTP.

HTTPS is required for WebGPU outside `localhost`; over plain HTTP on other hosts, SoundCraft falls back to WebGL2. HTTPS also provides the secure context required by browser clipboard features.

Do not expose an unencrypted HTTP port directly to the public Internet. Restrict access to your intended network or protect external access with HTTPS and appropriate access controls.

SoundCraft's web build uses WebAudio and does not require `SharedArrayBuffer`, `Cross-Origin-Opener-Policy` or `Cross-Origin-Embedder-Policy` isolation headers.

## Automated builds

GitHub Actions checks the official [SoundCraft releases](https://github.com/storytold/soundcraft/releases), builds the WebAssembly application from the selected upstream tag and publishes the image to GHCR.

| Trigger | Behaviour |
|---|---|
| Weekly schedule | Checks the latest official SoundCraft release and builds it if it has not already been published here |
| Manual workflow dispatch | Builds the latest official release, or a specified `upstream_tag` |
| Force rebuild | Rebuilds an already published version with `force_rebuild: true` |
| Changes to packaging files on `main` | Rebuilds the current official upstream release with the updated workflow |

### Published tags

- `latest` — the latest successfully published image (mutable tag).
- `<upstream-version>` — the exact official upstream tag, for example `vX.Y.Z`.
- `upstream-<commit>` — the upstream source commit used for the image (first 12 hexadecimal characters).

The image targets both `linux/amd64` and `linux/arm64`. The workflow checks out **the official upstream release tag**, rather than compiling `main` as the versioned source.

### GitHub Releases

After a successful Docker build and GHCR push, the workflow creates or updates a [GitHub Release](https://github.com/MikaPST/soundcraft-web/releases) matching the upstream tag. It includes the upstream release notes, source link and commit, Docker tags, supported architectures and (when available) the published image digest.

The Release badge at the top of this README updates automatically. Scheduled runs skip versions already released here. You can force a rebuild from **Actions → Build and publish SoundCraft Web → Run workflow**.

> [!IMPORTANT]
> A GitHub Release is created only after the Docker image has been successfully published. The `latest` tag may change after a rebuild; pin a versioned image tag for a stable deployment. Auxiliary source repositories (for example `craft-fonts`) may be checked out separately and are not necessarily pinned to the upstream release commit.

## Unraid Community Applications

This image is intended to be distributed through the maintainer's existing Unraid Community Applications repository.

The Unraid template references the published GHCR image and provides configuration for the web port.

- **Unraid Community Applications repository:** [MikaPST/unraid-community-apps](https://github.com/MikaPST/unraid-community-apps)
- **SoundCraft Web template:** to be added when the template is published.

This repository contains the image build, publication workflow and Docker Compose configuration. The Unraid Community Applications template is maintained separately.
- **Template:** [soundcraft-web.xml](https://github.com/MikaPST/unraid-community-apps/blob/main/templates/soundcraft-web.xml)

## Upstream project

SoundCraft is an open-source digital audio workstation (DAW), built in pure Rust by the ArtCraft Team and its contributors. It provides a multitrack editing and mixing workflow, with audio and MIDI tracks, automation, built-in effects and instruments, and audio export in the native application.

- **Source code:** [storytold/soundcraft](https://github.com/storytold/soundcraft)
- **Official website:** [getartcraft.com/apps/soundcraft](https://getartcraft.com/apps/soundcraft)
- **Official web hosting guide:** [Hosting SoundCraft for the web](https://github.com/storytold/soundcraft/blob/main/packaging/web/README.md)
- **Official releases:** [SoundCraft releases](https://github.com/storytold/soundcraft/releases)

The upstream web application is built with Trunk from `apps/soundcraft-web`. This repository builds that browser application from upstream source and packages the generated static files in an NGINX image. It does not modify the SoundCraft application itself.

The MikaPST GitHub Release tracks the upstream version but represents independent Docker packaging, not an official SoundCraft distribution.

## License and attribution

SoundCraft is dual-licensed under the **MIT License or Apache License 2.0**, at the user's option. Copyright belongs to the ArtCraft Team and SoundCraft contributors.

The upstream project includes license and attribution information for the application and bundled third-party components. These notices and the applicable license files must be preserved when redistributing the application.

The Docker packaging files in this repository are maintained separately from the upstream application. This repository does not claim ownership of SoundCraft or its bundled assets.

## Disclaimer

This is an independent community packaging project and is not affiliated with, endorsed by, or sponsored by the ArtCraft Team.

SoundCraft is provided by its upstream authors under its own license and development status. Use this Docker image at your own risk and consult the official repository for the latest information, limitations and notices.
