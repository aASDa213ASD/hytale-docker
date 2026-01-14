Docker setup for running the Hytale dedicated server.

This container downloads/updates the server on startup using the official Hytale Downloader CLI, then starts `HytaleServer.jar` with the options configured via environment variables.

Once the server runs you have to authorize with Hytale so client connects work.
Therefore you have to attach to the container `docker attach CONTAINER_NAME` if you use the default docker-compose.yml it's `hytale` and run `auth login device` you will the an authentication link navigating you to the hytale login page, once authenticated clients can connect. To save the authorization so you dont have to do it every restart you can run `auth persistence Encrypted` so it gets saved to an encrypted file.

The command `auth persistence Encrypted` needs the hardware id to be present, see the docker-compose.yml and uncomment the mapping of the hardware id (if your host system supports it, not every linux distro has the machine-id at the same location)

## Ports

- UDP `5520` (default)

## Environment variables

These are set in `docker-compose.yml`. You can change them there or move them into a `.env` and reference them from Compose.

### Server / networking

- `HYTALE_PORT` (default: `5520`)
  - Server port.
- `BIND_ADDR` (default: `0.0.0.0`)
  - Bind address.

### Assets / auth

- `ASSETS_PATH` (default: `/hytale/Assets.zip`)
  - Path to the server assets zip inside the container.
- `AUTH_MODE` (default: `authenticated`)
  - Authentication mode passed to the server.

#### Server Provider Authentication

You can pass auth tokens to the server. These are **not set by default** and the startup script only adds the flags when the variables are present:

- `SESSION_TOKEN` (optional)
  - Adds `--session-token "<sessionToken>"`
- `IDENTITY_TOKEN` (optional)
  - Adds `--identity-token "<identityToken>"`
- `OWNER_UUID` (optional)
  - Adds `--owner-uuid "<uuid>"`

### Updates

- `ENABLE_AUTO_UPDATE` (default: `true`)
  - If `true`, runs `./hytale-downloader` on every container start.
- `SKIP_DELETE_ON_FORBIDDEN` (default: `false`)
  - If `true`, the startup script will **not** delete `~/.hytale-downloader-credentials.json` when it detects a `403 Forbidden`.

### AOT cache

- `USE_AOT_CACHE` (default: `true`)
  - If `true` and `/hytale/Server/HytaleServer.aot` exists, adds `-XX:AOTCache=HytaleServer.aot`.

### Flags

- `ACCEPT_EARLY_PLUGINS` (default: `false`)
  - Adds `--accept-early-plugins`.
- `ALLOW_OP` (default: `false`)
  - Adds `--allow-op`.
- `DISABLE_SENTRY` (default: `false`)
  - Adds `--disable-sentry`.

### Backups

- `BACKUP_ENABLED` (default: `false`)
  - If `true`, adds backup arguments.
- `BACKUP_DIR` (default: `/hytale/backups`)
  - Backup output directory inside the container.
- `BACKUP_FREQUENCY` (default: `30`)
  - Backup frequency.

### Java memory

- `JAVA_XMS` (default: `4G`)
  - Adds `-Xms`.
- `JAVA_XMX` (default: `4G`)
  - Adds `-Xmx`.
- `JAVA_CMD_ADDITIONAL_OPTS` (optional)
  - Appends additional JVM args to the `java` command.


### Additional server options

- `HYTALE_ADDITIONAL_OPTS` (default: empty)
  - Appends additional **HytaleServer.jar** options after all other generated args.
  - Usable for server options which are not currently available as env variables.

Using `HYTALE_ADDITIONAL_OPTS` you can use any of the below commands to add it to the server startup command. It's recommended not to duplicate commands either by adding them multiple times or using the above env variables because of possible unknown issues.

Commands available as seen with `java -jar HytaleServer.jar --help`:

| Option | Description |
|---|---|
| `--accept-early-plugins` | You acknowledge that loading early plugins is unsupported and may cause stability issues. |
| `--allow-op` |  |
| `--assets <Path>` | Asset directory (default: ../HytaleAssets) |
| `--auth-mode <authenticated\|offline\|insecure>` | Authentication mode (default: AUTHENTICATED) |
| `-b, --bind <InetSocketAddress>` | Port to listen on (default: 0.0.0.0/0.0.0.0:5520) |
| `--backup` |  |
| `--backup-dir <Path>` |  |
| `--backup-frequency <Integer>` | (default: 30) |
| `--backup-max-count <Integer>` | (default: 5) |
| `--bare` | Runs the server bare, e.g. without loading worlds, binding to ports or creating directories. (Note: Plugins will still be loaded and may not respect this flag.) |
| `--boot-command <String>` | Runs command on boot. If multiple commands are provided they are executed synchronously in order. |
| `--client-pid <Integer>` |  |
| `--disable-asset-compare` |  |
| `--disable-cpb-build` | Disables building of compact prefab buffers |
| `--disable-file-watcher` |  |
| `--disable-sentry` |  |
| `--early-plugins <Path>` | Additional early plugin directories to load from |
| `--event-debug` |  |
| `--force-network-flush <Boolean>` | (default: true) |
| `--generate-schema` | Causes the server generate schema, save it into the assets directory and then exit |
| `--help` | Print's this message. |
| `--identity-token <String>` | Identity token (JWT) |
| `--log <KeyValueHolder>` | Sets the logger level. |
| `--migrate-worlds <String>` | Worlds to migrate |
| `--migrations <Object2ObjectOpenHashMap>` | The migrations to run |
| `--mods <Path>` | Additional mods directories |
| `--owner-name <String>` |  |
| `--owner-uuid <UUID>` |  |
| `--prefab-cache <Path>` | Prefab cache directory for immutable assets |
| `--session-token <String>` | Session token for Session Service API |
| `--shutdown-after-validate` | Automatically shutdown the server after asset and/or prefab validation. |
| `--singleplayer` |  |
| `-t, --transport <TransportType>` | Transport type (default: QUIC) |
| `--universe <Path>` |  |
| `--validate-assets` | Causes the server to exit with an error code if any assets are invalid. |
| `--validate-prefabs [ValidationOption]` | Causes the server to exit with an error code if any prefabs are invalid. |
| `--validate-world-gen` | Causes the server to exit with an error code if default world gen is invalid. |
| `--version` | Prints version information. |
| `--world-gen <Path>` | World gen directory |


## Notes

- If the downloader fails with a `403 Forbidden`, the startup script clears `~/.hytale-downloader-credentials.json` and retries on next start.

## Docs

- Server Provider Authentication Guide: https://support.hytale.com/hc/en-us/articles/45328341414043-Server-Provider-Authentication-Guide
- Hytale Server Manual: https://support.hytale.com/hc/en-us/articles/45326769420827-Hytale-Server-Manual
