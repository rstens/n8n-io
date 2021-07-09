FROM registry.redhat.io/rhel7

ARG N8N_VERSION

ENV TZ America/Vancouver
ENV SUMMARY="n8n-io" \
    DESCRIPTION="n8n-io"

LABEL summary=$SUMMARY \
      description="$DESCRIPTION" \
      io.k8s.description="$DESCRIPTION" \
      io.k8s.display-name="" \
      io.openshift.expose-services="" \
      io.openshift.tags="" \
      name="rhel7" \
      com.redhat.license_terms="https://www.redhat.com/en/about/red-hat-end-user-license-agreements#rhel" 

# Copy entitlements and subscription manager configurations
# https://github.com/BCDevOps/OpenShift4-Migration/issues/15
# COPY ./etc-pki-entitlement /etc/pki/entitlement
# COPY ./rhsm-conf /etc/rhsm
# COPY ./rhsm-ca /etc/rhsm/ca
#

RUN if [ -z "$N8N_VERSION" ] ; then echo "The N8N_VERSION argument is missing!" ; exit 1; fi
RUN curl -sL https://rpm.nodesource.com/setup_14.x | bash -

# We need to call 2 (!) yum commands before being able to enable repositories properly
# This is a workaround for https://bugzilla.redhat.com/show_bug.cgi?id=1479388
# Initialize /etc/yum.repos.d/redhat.repo
# See https://access.redhat.com/solutions/1443553
RUN rm /etc/rhsm-host && \
    yum repolist > /dev/null && \
    yum install -y yum-utils gettext && \
    yum-config-manager --disable \* &> /dev/null && \
    yum-config-manager --enable rhel-server-rhscl-7-rpms && \
    yum-config-manager --enable rhel-7-server-rpms && \
    yum-config-manager --enable rhel-7-server-optional-rpms && \
    INSTALL_PKGS="gcc-c++ make rsync tar gettext bind-utils nss_wrapper nodejs" && \
    yum -y --setopt=tsflags=nodocs install $INSTALL_PKGS && \
    rpm -V $INSTALL_PKGS && \
    yum -y clean all --enablerepo='*' && \
    ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone && \
    localedef -f UTF-8 -i en_US en_US.UTF-8

VOLUME ["/data"]

# Remove entitlements and Subscription Manager configs
#RUN rm -rf /etc/pki/entitlement && \
#    rm -rf /etc/rhsm

USER 1001

RUN npm_config_user=root npm install -g n8n@${N8N_VERSION}

WORKDIR /data

CMD "n8n"
