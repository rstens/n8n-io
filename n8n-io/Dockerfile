FROM  registry.access.redhat.com/rhel7:latest

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

COPY ./etc-pki-entitlement /etc/pki/entitlement

# Copy subscription manager configurations
COPY ./rhsm-conf /etc/rhsm
COPY ./rhsm-ca /etc/rhsm/ca


RUN if [ -z "$N8N_VERSION" ] ; then echo "The N8N_VERSION argument is missing!" ; exit 1; fi
RUN curl -sL https://rpm.nodesource.com/setup_12.x | bash -

RUN INSTALL_PKGS="gettext gcc-c++ make nodejs" && \
    rm /etc/rhsm-host && \
    yum repolist --disablerepo=* && \
    yum -y --setopt=tsflags=nodocs install $INSTALL_PKGS && \
    rpm -V $INSTALL_PKGS && \
    yum -y clean all --enablerepo='*' && \
    npm_config_user=root npm install request -g && \
    npm_config_user=root npm install -g n8n@${N8N_VERSION} && \
    ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone && \
    localedef -f UTF-8 -i en_US en_US.UTF-8

VOLUME ["/data"]
WORKDIR /data

# Remove entitlements and Subscription Manager configs
RUN rm -rf /etc/pki/entitlement && \
    rm -rf /etc/rhsm

RUN mkdir -p /.n8n && \
    chgrp -R 0 /data && \
    chmod -R g=u /data && \
	chgrp -R 0 /home && \
    chmod -R g=u /home && \
	chgrp -R 0 /usr && \
    chmod -R g=u /usr && \
	chgrp -R 0 /.n8n && \
    chmod -R g=u /.n8n && \
    chmod g=u /etc/passwd

EXPOSE 5678/tcp
ENTRYPOINT [ "uid_entrypoint" ]

USER 1001

CMD "n8n"
