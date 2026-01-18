# About this fork
When trying to use Octoprint on my Raspberry with Obico, I found out that the webcam stream was not available in Obico, which made the endeavour useless.

I found out that Obico needs Debian 11 (or Ubuntu 11), while the original docker container of Octoprint still uses Ubuntu 10 and thus no support for Obico. So I made the following changes to the Dockerfile:

- Changed Python base from 3.10-slim-bullseye to 3.10.19-slim-bookworm
- Change S6-overlay from v2.1.0.0 to v3.2.1.0 (the old version did not work with bookworm)
- Adapted the URLs for S6-overlay, as the packages have now been split up
- Added Janus package for streaming
- Created a fix-attrs file as S6 now had problems on running scripts on the container

## Building the image

On your machine, simply build the image with:

```
docker build -t my/octoprint .
```

I also exposed port 8080 to port 80, it seems that Obico needs that port for streaming (I think that can be adapted somewhere, but that's another story).

In the docker-compose.yml, I also changed the image name, so you can fire up the `docker compose up -d` as with the original.

## Further adaptions needed

### Octoprint restart
In Octoprint, the restart command has to be changed. To do this, go to Settings/Server/Commands and change "Restart OctoPrint" to

```
s6-svc -r /var/run/service/octoprint
```

### Obico Plugin
When using the container, the `/octoprint` volume is mounted with the `noexec` attrbute, and running the mpeg probe script of the plugin will fail with "permission denied".

To solve this, I simple entered the Docker container changed the following line in `/plugins/lib/python3.10/site-packages/octoprint_obico/webcam_stream.py`:

from
```
FFMPEG = os.path.join(FFMPEG_DIR, 'run.sh')
```

to

```
FFMPEG = '/usr/bin/bash ' + os.path.join(FFMPEG_DIR, 'run.sh')
```
(notice the space after `bash`).

OctoPrint still complains about ffmpeg in the logs, but the stream worked for me now.


# OctoPrint-docker (original docu)

Please find the original docu at 
https://github.com/OctoPrint/octoprint-docker/blob/master/README.md

I have no affiliations with the original maintainers, so for everything that is not 100% linked to my fork, please contact them.

I also don't have to capacity to test this image on all platforms, I only ran it on x86_64 and RasperryPi.