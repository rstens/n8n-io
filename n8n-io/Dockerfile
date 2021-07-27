FROM registry.access.redhat.com/ubi8/nodejs-12:latest

ARG N8N_VERSION

ENV HOME=/opt/app-root/src \
    TZ=America/Vancouver

RUN mkdir -p $HOME

WORKDIR $HOME

# If you are building your code for production
# RUN npm install --only=production
RUN npm install request -g && \
    npm install -g n8n@${N8N_VERSION}

ENV PATH ${HOME}/node_modules/.bin/:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:$PATH

VOLUME ${HOME}
VOLUME [ "/data" ]
WORKDIR /data

EXPOSE 5678/tcp

CMD ["n8n"]
