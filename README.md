# docker-images

Some useful public Docker images.
Mostly generated by prompting and re-prompting Claude Sonnet 3.5 till the results are passable.

## `node-python-ffmpeg`

A fat container with a fully-loaded FFMpeg 6 installation, Node.js latest, and Python 3.

* Node: 22.9.0
* Python: 3.10
* FFMpeg: 6.1.2 (with AV1, Opus support)

To run with NVidia GPU support add `--gpus all` to `docker run`.
