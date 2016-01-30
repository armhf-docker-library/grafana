FROM armhfbuild/golang:1.5

# install node:5.5.0
# see https://github.com/armhf-docker-library/node/blob/master/5.5.0/Dockerfile

# gpg keys listed at https://github.com/nodejs/node
RUN set -ex \
  && for key in \
    9554F04D7259F04124DE6B476D5A82AC7E37093B \
    94AE36675C464D64BAFA68DD7434390BDBE9B9C5 \
    0034A06D9D9B0064CE8ADF6BF1747F4AD2306D93 \
    FD3A5288F042B6850C66B31F09FE44734EB7990E \
    71DCFD284A79C3B38668286BC97EC7A07EDE3FC1 \
    DD8F2338BAE7501E3DD5AC78C273792F7D83545D \
    B9AE9905FFD7803F25714661B63B535A4C206CA9 \
    C4F0DFFF4E8C1A8236409D08E73BC641CC11F4C8 \
  ; do \
    gpg --keyserver ha.pool.sks-keyservers.net --recv-keys "$key"; \
  done

ENV NPM_CONFIG_LOGLEVEL info
ENV NODE_VERSION 5.5.0

RUN arch="armv7l" \
  && curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-$arch.tar.gz" \
  && curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc" \
  && gpg --verify SHASUMS256.txt.asc \
  && grep " node-v$NODE_VERSION-linux-$arch.tar.gz\$" SHASUMS256.txt.asc | sha256sum -c - \
  && tar -xzf "node-v$NODE_VERSION-linux-$arch.tar.gz" -C /usr/local --strip-components=1 \
  && rm "node-v$NODE_VERSION-linux-$arch.tar.gz" SHASUMS256.txt.asc

# grab gosu for easy step-down from root
RUN gpg --keyserver ha.pool.sks-keyservers.net --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4
RUN arch="$(dpkg --print-architecture)" \
	&& set -x \
	&& curl -o /usr/local/bin/gosu -SL "https://github.com/tianon/gosu/releases/download/1.7/gosu-$arch" \
	&& curl -o /usr/local/bin/gosu.asc -SL "https://github.com/tianon/gosu/releases/download/1.7/gosu-$arch.asc" \
	&& gpg --verify /usr/local/bin/gosu.asc \
	&& rm /usr/local/bin/gosu.asc \
	&& chmod +x /usr/local/bin/gosu

RUN go get github.com/grafana/grafana \
  && cd $GOPATH/src/github.com/grafana/grafana \
  && go run build.go setup \
  && godep restore \
  && go run build.go build

RUN cd $GOPATH/src/github.com/grafana/grafana \
  && sed -i '/phantomjs/d' package.json \
  && npm install \
  && npm install -g grunt-cli \
  && grunt --force

RUN groupadd -r grafana && useradd -r -m -g grafana grafana

RUN cd $GOPATH/src/github.com/grafana/grafana \
  && cp bin/grafana-server /usr/sbin/ \
  && mkdir -p /usr/share/grafana /etc/grafana /var/lib/grafana /var/log/grafana \
  && cp -r conf /usr/share/grafana/ \
  && cp -r public_gen/ /usr/share/grafana/public

WORKDIR /
RUN rm -r /go

VOLUME ["/var/lib/grafana", "/var/log/grafana", "/etc/grafana"]
EXPOSE 3000
COPY ./run.sh /run.sh
COPY ./grafana.ini /etc/grafana/
ENTRYPOINT ["/run.sh"]
