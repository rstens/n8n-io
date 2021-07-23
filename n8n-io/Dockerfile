FROM  image-registry.openshift-image-registry.svc:5000/af2668-tools/rhel7

ARG N8N_VERSION=0.36.1

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
RUN curl -sL https://rpm.nodesource.com/setup_14.x | bash -

RUN INSTALL_PKGS="gcc-c++ make nodejs" && \
    rm /etc/rhsm-host && \
    yum repolist --disablerepo=* && \
    yum -y --setopt=tsflags=nodocs install $INSTALL_PKGS && \
    rpm -V $INSTALL_PKGS
    
RUN yum -y clean all --enablerepo='*' && \
    ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone && \
    localedef -f UTF-8 -i en_US en_US.UTF-8

VOLUME ["/data"]
WORKDIR /data

# Remove entitlements and Subscription Manager configs
RUN rm -rf /etc/pki/entitlement && \
    rm -rf /etc/rhsm

RUN chown -R 1001:0 /data

USER 1001

RUN npm_config_user=1001 npm install -g n8n@${N8N_VERSION}

EXPOSE 5678/tcp

CMD "n8n"
