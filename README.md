# Altserver-docker for Synology

This repo is a fork of the original [dreth/Altserver-docker](https://github.com/dreth/Altserver-docker) repo which did the heavy lifting for this set-up. The original README.md has been preserved [here](./Altserver-docker.README.md) to retain the original instructions for general use. The below instructions are an abridged version of what I did to make this work on my Synology DS218+ running DSM 7.1

If you already have the original repo running, **tldr**:

> Override the `supervisord.conf` in the Docker image with the updated version in ([./conf](./conf)) by adding `"./conf:/etc/supervisor/conf.d"` to your volumes in [docker-compose.yml](./docker-compose.yml) to disable `usbmuxd` in the container.

---

## Requirements

- Docker package

## Installation

1. Run original `docker-compose.yml` - [how to](#run-using-docker-compose-recommended)
2. Connect device via USB
   - Trust device
   - Install AltStore - [how to](#install-altstore-on-ios-device)
3. Update files as such
   - `docker-compose.yml`
   ```yml
   volumes:
     - "./conf:/etc/supervisor/conf.d/" # Add this volume
   ```
   - `supervisord.conf` - delete the section for `usbmuxd`
4. Run updated `docker-compose.yml` - [how to](#run-using-docker-compose-recommended)

## Run Using docker-compose (recommended)

To start up the application, you can run:

```shell
docker compose up -d
```

Or optionally if you'd like to build it yourself, modify the docker-compose.yml, uncomment the build config and run the docker-compose stack with the optional build flag:

```shell
docker compose up -d --build
```

## Install AltStore on iOS Device

1. Make sure you have your Device UDID (can be found using [this guide](https://discussions.apple.com/thread/250783627)).

2. Run this command to install AltStore to your device:

```shell
docker exec -it altserver /altserver/bin/AltServer -u "<UDID>" -a "<Apple ID>" -p "<Password>" /altserver/bin/AltStore.ipa
```

## Logs

Logs will be stored in the directory where the container is ran inside `./logs`

## Optional environment variables

### Overriding architectures for libraries and binaries

It's possible to override which architecture of altserver, netmuxd, anisette-server and provision libraries (for anisette) are downloaded by setting the following environment variables in the docker compose file:

```yaml
environment:
  - OVERRIDE_ALTSERVER_ARCH=x86_64
  - OVERRIDE_NETMUXD_ARCH=x86_64
  - OVERRIDE_ANISETTE_ARCH=x86_64
  - OVERRIDE_PROVISION_LIBS_ARCH=x86_64
```

You can check for which architectures are available by checking the releases of each project:

- [AltServer-Linux](https://github.com/NyaMisty/AltServer-Linux/releases)
- [netmuxd](https://github.com/jkcoxson/netmuxd/releases)
- [Provision](https://github.com/Dadoum/Provision/releases) (this is the anisette-server)
- The libraries are pulled from the apple music android apk, so the only available options are `arm64-v8a`, `armeabi-v7a`, `x86`, `x86_64`

### Overriding whether provision libraries are decompressed from the repo or downloaded

It's possible to allow provision to download the libraries it needs from the apple music android apk by setting the following environment variable in the docker compose file:

```yaml
environment:
  - ALLOW_PROVISION_TO_DOWNLOAD_LIBS=true
```

When this variable is present, the libraries will be downloaded by provision, if it's not present, the libraries will be decompressed from the repo.

## Provision libraries

Provision automatically pulls the libraries it requires (libCoreADI.so and libstoreservicescore.so) from the apple music android apk, but this requires a 60+MB download it does automatically, so I decided to include these in the root of the repo already and have the `docker-entrypoint.sh` decompress them where Provision normally would. This is optional and can be overridden by setting the `ALLOW_PROVISION_TO_DOWNLOAD_LIBS` environment variable to `true`.

### Updating lib.tar.xz

To update lib.tar.xz just run:

```shell
bash scripts/provision-deps-manual-download.sh
```

from the root of the repo.

## Credits

I have only compiled these instructions to specifically run this on Synology, the original author deserves the credit for originally containerising and automating the configuration of the applications, and the original authors of the applications the container is dependent on.

Containerising

- [Altserver-docker](https://github.com/dreth/Altserver-docker)

Applications

- [AltServer-Linux](https://github.com/NyaMisty/AltServer-Linux)
- [netmuxd](https://github.com/jkcoxson/netmuxd/)
- [Provision](https://github.com/Dadoum/Provision/)
- [Altstore](https://altstore.io)
