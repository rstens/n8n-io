FROM rhscl/s2i-base-rhel7:1

ARG N8N_VERSION
EXPOSE 5678

# This image will be initialized with "npm run $NPM_RUN"
# See https://docs.npmjs.com/misc/scripts, and your repo's package.json
# file for possible values of NPM_RUN
# Description
# Environment:
# * $NPM_RUN - Select an alternate / custom runtime mode, defined in your package.json files' scripts section (default: npm run "start").

ENV TZ=America/Vancouver \
    NODEJS_VERSION=12 \
    NPM_RUN=start \
    NAME=nodejs \
    NPM_CONFIG_PREFIX=$HOME/.npm-global

# Set SCL related variables in Dockerfile so that the collection is enabled by default
# Add $HOME/node_modules/.bin to the $PATH, allowing user to make npm scripts
# available on the CLI without using npm's --global installation mode
ENV SUMMARY="Platform for building and running Node.js $NODEJS_VERSION applications" \
    DESCRIPTION="Node.js $NODEJS_VERSION available as container is a base platform for \
building and running various Node.js $NODEJS_VERSION applications and frameworks. \
Node.js is a platform built on Chrome's JavaScript runtime for easily building \
fast, scalable network applications. Node.js uses an event-driven, non-blocking I/O model \
that makes it lightweight and efficient, perfect for data-intensive real-time applications \
that run across distributed devices." \
    PATH=/opt/rh/rh-nodejs$NODEJS_VERSION/root/usr/bin:$HOME/node_modules/.bin/:$HOME/.npm-global/bin/:$PATH \
    LD_LIBRARY_PATH=/opt/rh/rh-nodejs$NODEJS_VERSION/root/usr/lib64 \
    X_SCLS=rh-nodejs$NODEJS_VERSION \
    MANPATH=/opt/rh/rh-nodejs$NODEJS_VERSION/root/usr/share/man \
    APP_ROOT=/opt/app-root \
    STI_SCRIPTS_PATH=/usr/libexec

LABEL summary="$SUMMARY" \
      description="$DESCRIPTION" \
      io.k8s.description="$DESCRIPTION" \
      io.k8s.display-name="Node.js $NODEJS_VERSION" \
      io.openshift.expose-services="8080:http" \
      io.openshift.tags="builder,$NAME,${NAME}${NODEJS_VERSION}" \
      io.openshift.s2i.scripts-url="image:///usr/libexec/s2i" \
      io.s2i.scripts-url="image:///usr/libexec/s2i" \
      com.redhat.dev-mode="DEV_MODE:false" \
      com.redhat.deployments-dir="${APP_ROOT}/src" \
      com.redhat.dev-mode.port="DEBUG_PORT:5858" \
      com.redhat.component="rh-${NAME}${NODEJS_VERSION}-container" \
      name="rhscl/$NAME-$NODEJS_VERSION-rhel7" \
      version="1" \
      com.redhat.license_terms="https://www.redhat.com/en/about/red-hat-end-user-license-agreements#UBI" \
      maintainer="SoftwareCollections.org <sclorg@redhat.com>" \
      help="For more information visit https://github.com/sclorg/s2i-nodejs-container" \
      usage="s2i build <SOURCE-REPOSITORY> rhscl/$NAME-$NODEJS_VERSION-rhel7:latest <APP-NAME>"

# Copy entitlements and subscription manager configurations
# https://github.com/BCDevOps/OpenShift4-Migration/issues/15

COPY ./etc-pki-entitlement /etc/pki/entitlement

# Copy subscription manager configurations
COPY ./rhsm-conf /etc/rhsm
COPY ./rhsm-ca /etc/rhsm/ca

RUN if [ -z "$N8N_VERSION" ] ; then echo "The N8N_VERSION argument is missing!" ; exit 1; fi

RUN rm /etc/rhsm-host && \
    yum install -y yum-utils && \
    prepare-yum-repositories rhel-server-rhscl-7-rpms && \
    ( [ "rh-${NAME}${NODEJS_VERSION}" != "${NODEJS_SCL}" ] && yum remove -y ${NODEJS_SCL}\* || : ) && \
    INSTALL_PKGS="rh-nodejs${NODEJS_VERSION} rh-nodejs${NODEJS_VERSION}-npm rh-nodejs${NODEJS_VERSION}-nodejs-nodemon nss_wrapper gettext gcc-c++ make" && \
    ln -s /usr/lib/node_modules/nodemon/bin/nodemon.js /usr/bin/nodemon && \
    yum install -y --setopt=tsflags=nodocs $INSTALL_PKGS && \
    rpm -V $INSTALL_PKGS && \
    yum -y clean all --enablerepo='*' && \
    npm install request -g && \
    npm install -g n8n@${N8N_VERSION}

# Copy the S2I scripts from the specific language image to $STI_SCRIPTS_PATH
COPY ./s2i/bin/ $STI_SCRIPTS_PATH

# Copy extra files to the image.
COPY ./root/ /

WORKDIR /data

# Remove entitlements and Subscription Manager configs
RUN rm -rf /etc/pki/entitlement && \
    rm -rf /etc/rhsm

# Drop the root user and make the content of /opt/app-root owned by user 1001
RUN chown -R 1001:0 ${APP_ROOT} && chmod -R ug+rwx ${APP_ROOT} && \
    rpm-file-permissions

USER 1001

CMD "n8n"
