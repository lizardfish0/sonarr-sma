# Official SMA container based on linuxserver/sonarr

Docker container for Sonarr that includes all FFMPEG and python requirements to run SMA with Sonarr.

## Version Tags

|Tag|Description|
|---|---|
|latest|Stable release from linuxserver/sonarr with FFMPEG compiled from linuxserver/ffmpeg|
|build|Stable release from linuxserver/sonarr with FFMPEG compiled from jrottenberg/ffmpeg|
|develop|Develop release from Sonarr with precompiled FFMPEG binaries|

## Usage

### Recent update
As of 3/9/2020 the containers were overhauled and the location of the script was changed from `/usr/local/bin/sma/sickbeard_mp4_automator` to `/usr/local/sma`. The autoProcess mount point has been modified as well to be more docker friendly in a `/usr/local/sma/config` directory. Please review and update accordingly.

### docker-compose
~~~yml
services:
  sonarr:
    image: mdhiggins/sonarr-sma
    container_name: sonarr
    volumes:
      - /opt/appdata/sonarr:/config
      - /opt/appdata/sma:/usr/local/sma/config
      - /mnt/storage/tv:/tv
      - /mnt/storage/downloads:/downloads
    ports:
      - 8989:8989
    restart: always
    environment:
      - PUID=${PUID}
      - PGID=${PGID}
~~~

### autoProcess.ini
- Mount autoProcess.ini containing directory to `/usr/local/sma/config` using volumes
 - Consider making this writable as new options will be auto written to the config as they are added
- Sonarr configuration options are read from `config.xml` inside the container and injected at runtime into `autoProcess.ini`
 - ffmpeg
 - ffprobe
 - host (read from environment variable or set to 127.0.0.1)
 - web_root
 - port
 - ssl

### Python Environment
The script installs all dependencies in a virtual environment, not the container/system level Python environment. In order to use the Python environment with the dependencies installed, please execute using `/usr/local/sma/venv/bin/python3`. Use this same Python executable if using manual.py

## Configuring Sonarr

###  Enable completed download handling
- Settings > Download Client > Completed Download Handling > Enable: Yes

### Add Custom Script
- Settings > Connect > + Add > Custom Script

|Parameter|Value|
|---|---|
|On Grab| No|
|On Download| Yes|
|On Upgrade| Yes|
|On Rename| No|
|Path|`/usr/local/sma/postSonarr.sh`|

**Make sure you're using the .sh file, no the .py file, the .sh file points to the appropriate virtual environment**

## Logs

Located at `/usr/local/sma/config/sma.log` inside the container and your mounted config folder

## Environment Variables
|Variable|Description|
|---|---|
|PUID|User ID|
|PGID|Group ID|
|HOST|Local IP address for callback requests, default `127.0.0.1`|
|SMA_PATH|`/usr/local/sma`|
|SMA_UPDATE|Default `false`. Set `true` to pull git update of SMA on restart|
|SMA_FFMPEG_URL|If provided, override linuxserver/ffmpeg with a static build provided by the URL.|
|SMA_STRIP_COMPONENTS|Default `1`. Number of components to strip from your tar.xz file when extracting so that FFmpeg binaries land in `/usr/local/bin`|

## Hardware Acceleration
The default image is built with [linuxserver/ffmpeg](https://hub.docker.com/r/linuxserver/ffmpeg), which supports VAAPI, QSV, and NVEnc/NVDec.

For VAAPI/QSV, you need to mount the hardware device from `/dev/dri`. 

Nvidia GPU support requires the `nvidia` runtime, available by installing [nvidia-container-toolkit](https://github.com/NVIDIA/nvidia-container-toolkit).

### VAAPI docker-compose sample
~~~yml
services:
  sonarr:
    container_name: sonarr
    devices:
      - /dev/dri/renderD128:/dev/dri/renderD128
~~~

### NVIDIA / NVEnc  NVDec docker-compose sample
~~~yml
services:
  sonarr:
    container_name: sonarr
    runtime: nvidia
~~~

## Custom Builds
Using the `build` tag leverages multi-stage docker builds to generate FFMPEG compiled using [jrottenberg/ffmpeg's](https://hub.docker.com/r/jrottenberg/ffmpeg) containers. This allows flexibility with building FFMPEG using special options such as VAAPI or NVENC (though these are now included by default in the default linuxserver/ffmpeg image). Building locally allows `ARG` values to be set to change the underlying parent container tags as below. It is recommended that you match your Ubuntu version in the ffmpeg_tag and sonarr_tag to ensure no missing dependencies.

|ARG|Default|Description|
|---|---|---|
|ffmpeg_tag|latest|Set tag to correspond to jrottenberg/ffmpeg:tag|
|sonarr_tag|latest|Set tag to correspond to linuxserver/sonarr:tag|
|extra_packages||Set additional packages/dependencies that might need to be installed via apt-get or apk, separated by spaces|