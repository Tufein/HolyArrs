# HolyArrs for umbrelOS

An [umbrelOS](https://umbrel.com) community app package for [HolyArrs](https://holyarrs.com), the self-hosted app that finds, downloads and organises movies, shows and music.

This repository is a community app store. It contains one app:

| App | Version | Upstream |
| --- | --- | --- |
| HolyArrs | 1.3.1 | [HolyArrs/HolyArrs](https://github.com/HolyArrs/HolyArrs) |

The app runs the official `holyarrs/holyarrs` image, pinned by tag and manifest-list digest. It listens on port `6262` behind umbrelOS's app proxy and keeps its state in the app's data folder.

**Store URL:** <https://github.com/Tufein/HolyArrs>

## Install

1. In umbrelOS, open **App Store**, open the **Community App Stores** dialog from the **⋯** menu, and add `https://github.com/Tufein/HolyArrs`.
2. Refresh the store. **HolyArrs** appears in the store; install it.
3. After installation, open the app's settings and choose a folder for **Media Library**. It is mounted at `/media` inside HolyArrs; until you choose one, `/media` points at the app's own data folder. The shared **Downloads** folder is already mounted at `/downloads`.
4. Open HolyArrs and create the owner account immediately. There is no default password; the first visitor becomes the owner.
5. In HolyArrs, add Library folders:
   - `/media/movies`
   - `/media/tv`
   - `/media/music`

   Create those subfolders in the media folder you selected, or point HolyArrs at folders that already exist inside it.
6. Connect a downloader and at least one source/indexer. HolyArrs works with Transmission, qBittorrent, SABnzbd, NZBGet and others, plus Plex, Jellyfin or Emby as the media server.

## Important: back up the master key

HolyArrs encrypts stored passwords and API tokens with `/config/holyarrs.key`. Umbrel app backups do **not** include this key.

As the owner, open **Settings → Backups → Download key kit** and store it outside the app. Without the key, a restored backup cannot decrypt its credentials.

## Folders and imports

| Umbrel | Inside HolyArrs | Notes |
| --- | --- | --- |
| Shared Downloads (`~/Downloads`) | `/downloads` | Default download folder, shared with Transmission and other Umbrel apps. |
| Your Media Library folder | `/media` | Selected in the app's settings; defaults to the app's data folder. Read-write, because HolyArrs moves imports here. |

Because umbrelOS mounts Downloads and the media library as two separate mounts, HolyArrs imports by **copy**, not by hardlink. The downloader keeps its own copy until it removes the torrent. This is normal for Umbrel apps; the alternative is a custom single-parent mount for both downloads and media, which this package does not do because it would bypass Umbrel's shared Downloads folder.

The default `data/media` folder is excluded from Umbrel app backups, so keep your own backup of the media library.

If HolyArrs cannot import from `/downloads`, add `/downloads` under **Settings → Library folders → Extra import folders**.

## Configuration

The app exposes one setting in the umbrelOS UI:

- `TZ` — time zone, default `Etc/UTC`, for example `Europe/Amsterdam`.

The compose file also passes:

- `PUID=1000` / `PGID=1000` — the Umbrel app user, matching the shared folders.
- `HOLYARRS_TRUSTED_PROXIES=${NETWORK_IP}/16` — the umbrelOS app network, so client IPs survive the proxy.
- `HOLYARRS_ALLOWED_HOSTS=${DEVICE_DOMAIN_NAME},${DEVICE_HOSTNAME}` — the normal `umbrel.local` names. IP addresses and `localhost` are accepted by HolyArrs out of the box.

## Updating

1. Check the upstream release and image digest:
   ```sh
   docker buildx imagetools inspect holyarrs/holyarrs:1.3.1
   ```
   Use the manifest-list digest shown for the release.
2. Update `version` in `holyarrs/umbrel-app.yml` and the `image` line in `holyarrs/docker-compose.yml`.
3. Add release notes to the manifest if you want them shown in the store.
4. Commit and push. In umbrelOS, refresh the community store and update the app.

Never remove `/config` during an update. The app stores its databases, settings, logs, backups and master key there.

## Repository layout

```
.
├── umbrel-app-store.yml       # store id + display name
├── README.md
└── holyarrs/
    ├── umbrel-app.yml         # app manifest
    ├── docker-compose.yml     # app proxy + HolyArrs server
    └── data/
        ├── config/            # tracked so the bind-mount folder exists
        └── media/             # default /media until a folder is selected
```

## Notes

- The app has no default credentials. The first person to open it becomes the owner, which is why it should be claimed right after installation.
- The container image has no shell. Use the app logs, `/config/logs/holyarrs.log`, or `docker exec holyarrs_server_1 /holyarrs doctor` for diagnostics.
- The image supports amd64, arm64 and arm/v7.
- HolyArrs is licensed under AGPL-3.0. This package only contains umbrelOS metadata and configuration.

## Validation

Before updating, check that all three YAML files parse and that the image tag and digest resolve. After installation, verify:

- The app reaches **running** and its health check passes.
- `http://umbrel.local:6262` (or the app tile) opens HolyArrs.
- Creating the owner account works.
- `/downloads` and `/media` are visible under **Settings → Library folders**.
- Restarting the app keeps the owner account and settings.
