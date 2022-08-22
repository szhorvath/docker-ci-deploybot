FROM --platform=linux/amd64 ubuntu:20.04

LABEL maintainer="Sandor Horvath"

ARG WWWGROUP
ARG NODE_VERSION=16

WORKDIR /var/www/html

ENV DEBIAN_FRONTEND noninteractive
ENV TZ=UTC

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN apt-get update \
    && apt-get install -y gnupg gosu curl ca-certificates zip unzip git supervisor sqlite3 libcap2-bin libpng-dev python2 \
    && mkdir -p ~/.gnupg \
    && chmod 600 ~/.gnupg \
    && echo "disable-ipv6" >> ~/.gnupg/dirmngr.conf \
    && echo "keyserver hkp://keyserver.ubuntu.com:80" >> ~/.gnupg/dirmngr.conf

# Node 16
RUN curl -sL https://deb.nodesource.com/setup_$NODE_VERSION.x | bash - \
    && apt-get install -y nodejs \
    && npm install -g npm

# Dependencies for running Cypress.io Test Runner
RUN apt-get update && \
    apt-get install --no-install-recommends -y \
    libgtk2.0-0 \
    libgtk-3-0 \
    libnotify-dev \
    libgconf-2-4 \
    libgbm-dev \
    libnss3 \
    libxss1 \
    libasound2 \
    libxtst6 \
    xauth \
    xvfb \
    # install text editors
    vim-tiny \
    nano \
    # install emoji font
    fonts-noto-color-emoji \
    # install Chinese fonts
    # this list was copied from https://github.com/jim3ma/docker-leanote
    fonts-arphic-bkai00mp \
    fonts-arphic-bsmi00lp \
    fonts-arphic-gbsn00lp \
    fonts-arphic-gkai00mp \
    fonts-arphic-ukai \
    fonts-arphic-uming \
    ttf-wqy-zenhei \
    ttf-wqy-microhei \
    xfonts-wqy \
    # clean up
    && rm -rf /var/lib/apt/lists/* \
    && apt-get clean

# avoid too many progress messages
# https://github.com/cypress-io/cypress/issues/1243
ENV CI=1 \
    # disable shared memory X11 affecting Cypress v4 and Chrome
    # https://github.com/cypress-io/cypress-docker-images/issues/270
    QT_X11_NO_MITSHM=1 \
    _X11_NO_MITSHM=1 \
    _MITSHM=0 \
    # point Cypress at the /root/cache no matter what user account is used
    # see https://on.cypress.io/caching
    CYPRESS_CACHE_FOLDER=/root/.cache/Cypress \
    # Allow projects to reference globally installed cypress
    NODE_PATH=/usr/local/lib/node_modules

# CI_XBUILD is set when we are building a multi-arch build from x64 in CI.
# This is necessary so that local `./build.sh` usage still verifies `cypress` on `arm64`.
ARG CI_XBUILD

RUN echo "whoami: $(whoami)" \
    && npm config -g set user $(whoami) \
    # command "id" should print:
    # uid=0(root) gid=0(root) groups=0(root)
    # which means the current user is root
    && id \
    && npm install -g typescript \
    && npm install -g "cypress@10.6.0" \
    # Cypress cache and installed version

    # give every user read access to the "/root" folder where the binary is cached
    # we really only need to worry about the top folder, fortunately
    && ls -la /root \
    && chmod 755 /root \
    # always grab the latest Yarn
    # otherwise the base image might have old versions
    # NPM does not need to be installed as it is already included with Node.
    # Show where Node loads required modules from
    && node -p 'module.paths'

# give every user read access to the "/root" folder where the binary is cached
# we really only need to worry about the top folder, fortunately
RUN ls -la /root
RUN chmod 755 /root

# Other
RUN mkdir ~/.ssh
RUN touch ~/.ssh_config

# Display versions installed
RUN node -v
RUN npm -v
