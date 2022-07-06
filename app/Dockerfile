FROM alpine:3.10

# Some ENV variables
ENV PATH="/mattermost/bin:${PATH}"
ENV MM_VERSION=5.23.1

# change this link
ARG WEB_APP_LINK=https://ib-com-backend-media-static.s3.ap-south-1.amazonaws.com/ib-com-webapp/ibcom-otg/ib-com-web-otg-beta-sso-v1.2-sv5.33.tar.gz

ARG SERVER_LINK=https://ib-com-backend-media-static.s3.ap-south-1.amazonaws.com/ib-com-server/ib-com-server-v1.0-sv5.33.tar.gz

# Build argument to set Mattermost edition
ARG edition=enterprise
ARG PUID=2000
ARG PGID=2000
ARG MM_BINARY=

# Install some needed packages
RUN apk add --no-cache \
    ca-certificates \
    curl \
    jq \
    libc6-compat \
    libffi-dev \
    libcap \
    linux-headers \
    mailcap \
    netcat-openbsd \
    xmlsec-dev \
    tzdata \
    && rm -rf /tmp/*

# Get Mattermost
RUN mkdir -p /mattermost/data /mattermost/plugins /mattermost/client/plugins \
    && if [ ! -z "$MM_BINARY" ]; then curl $MM_BINARY | tar -xvz ; \
    elif [ "$edition" = "team" ] ; then curl $SERVER_LINK | tar -xvz ; \
    else curl $SERVER_LINK | tar -xvz ; fi \
    && cp /mattermost/config/config.json /config.json.save \
    && rm -rf /mattermost/config/config.json \
    && addgroup -g ${PGID} mattermost \
    && adduser -D -u ${PUID} -G mattermost -h /mattermost -D mattermost \
    && chown -R mattermost:mattermost /mattermost /config.json.save /mattermost/plugins /mattermost/client/plugins \
    && setcap cap_net_bind_service=+ep /mattermost/bin/mattermost

RUN cd /mattermost \
    && rm -rf client \
    && curl $WEB_APP_LINK | tar -xvz \
    && mkdir /mattermost/client/plugins \
    && chown -R mattermost:mattermost /mattermost/client

RUN cd /mattermost/i18n \
    && rm -rf *.json \
    && wget https://ibcom-storage.s3.ap-south-1.amazonaws.com/web/i18n.tar \
    && tar -xf i18n.tar \
    && rm i18n.tar \
    && chown -R mattermost:mattermost /mattermost/i18n

RUN cd /mattermost/templates \
    && rm -rf *.html \
    && wget https://ibcom-storage.s3.ap-south-1.amazonaws.com/web/templates.tar \
    && tar -xf templates.tar \
    && rm templates.tar \
    && chown -R mattermost:mattermost /mattermost/templates

USER mattermost

#Healthcheck to make sure container is ready
HEALTHCHECK CMD curl --fail http://localhost:8000 || exit 1

# Configure entrypoint and command
COPY entrypoint.sh /
ENTRYPOINT ["/entrypoint.sh"]
WORKDIR /mattermost
CMD ["mattermost"]

# Expose port 8000 of the container
EXPOSE 8000

# Declare volumes for mount point directories
VOLUME ["/mattermost/data", "/mattermost/logs", "/mattermost/config", "/mattermost/plugins", "/mattermost/client/plugins"]
