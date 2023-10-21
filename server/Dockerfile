FROM rocker/r-ver:4.3

RUN apt-get update && apt-get install --no-install-recommends -y \
    nano \
    libcurl4-openssl-dev \
    openssh-client \
    less \
    htop \
    procps \
    net-tools \
    rsync \
    gdebi-core \
    libxt-dev \
    man-db && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

# Setup various variables
ENV TZ="Europe/Helsinki" \
    HOME="/mnt/${NAME}-pvc" \
    TINI_VERSION=v0.18.0 \
    APP_UID=999 \
    APP_GID=999 \
    PKG_R_VERSION=3.4.4 \
    PKG_RSTUDIO_VERSION=1.1.447 \
    PKG_SHINY_VERSION=1.5.7.907

# Setup Tini, as S6 does not work when run as non-root users
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini /sbin/tini
RUN chmod +x /sbin/tini

RUN apt-get -y update
RUN apt-get -y install git coreutils wget cmake


# Clone the repository from GitHub
RUN git clone https://github.com/rstudio/shiny-server.git

# Get into a temporary directory in which we'll build the project
WORKDIR ./shiny-server
RUN mkdir tmp
WORKDIR ./tmp


RUN rm /bin/sh && ln -s /bin/bash /bin/sh

# Install our private copy of Node.js
# RUN ../external/node/install-node.sh
ENV NVM_DIR /usr/local/nvm
ENV NODE_VERSION 21.0.0
RUN mkdir -p /usr/local/nvm
RUN wget https://raw.githubusercontent.com/nvm-sh/nvm/v0.37.2/install.sh
RUN chmod +x ./install.sh
RUN ./install.sh


RUN source $NVM_DIR/nvm.sh \
    && nvm install $NODE_VERSION \
    && nvm alias default $NODE_VERSION \
    && nvm use default




# Add the bin directory to the path so we can reference node
ENV DIR=`pwd`
ENV PATH=$DIR/../bin:$PATH

# Use cmake to prepare the make step. Modify the "--DCMAKE_INSTALL_PREFIX"
# if you wish the install the software at a different location.
RUN cmake -DCMAKE_INSTALL_PREFIX=/usr/local ../
# Get an error here? Check the "How do I set the cmake Python version?" question below

# Compile native code and install npm dependencies
RUN make
RUN mkdir ../build
# Bir üst dizine çık ve npm install çalıştır
WORKDIR ../
# add node and npm to path so the commands are available
ENV NODE_PATH $NVM_DIR/v$NODE_VERSION/lib/node_modules
ENV PATH $NVM_DIR/versions/node/v$NODE_VERSION/bin:$PATH
RUN npm install


WORKDIR ./tmp
# Install the software at the predefined location
# Burada tekrar tmp dizinine dönmek gerekli
RUN make install

# Install default config file
RUN mkdir -p /etc/shiny-server
RUN cp ../config/default.config /etc/shiny-server/shiny-server.conf


# Place a shortcut to the shiny-server executable in /usr/bin
RUN ln -s /usr/local/shiny-server/bin/shiny-server /usr/bin/shiny-server

# Create shiny user. On some systems, you may need to specify the full path to 'useradd'
RUN useradd -r -m shiny

# Create log, config, and application directories
RUN mkdir -p /var/log/shiny-server
RUN mkdir -p /srv/shiny-server
RUN mkdir -p /var/lib/shiny-server
RUN chown shiny /var/log/shiny-server
RUN mkdir -p /etc/shiny-server


RUN chown 999.999 /var/log/shiny-server && chmod go+w -R /var/log/shiny-server /usr/local/lib/R /srv /var/lib/shiny-server
