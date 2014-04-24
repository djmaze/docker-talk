Introductory docker talk by me.

## Run this presentation:

First [install docker](https://www.docker.io/gettingstarted/#h_installation).

Afterwards, run:

    sudo docker run -d -p 8000:8000 mazzolino/docker-talk

Then visit [http://localhost:8000](http://localhost:8000) (or something different if using [boot2docker](https://github.com/boot2docker/boot2docker)).

## Develop

if you checked out the source code repository, you can use [fig](http://orchardup.github.io/fig/) to build & run the container.

Just run:

    sudo fig up

## Copyright

This work is licensed under a Creative Commons Attribution-ShareAlike 4.0 International License.

I could not distribute all images because they might be copyrighted. So I linked to them accordingly.
