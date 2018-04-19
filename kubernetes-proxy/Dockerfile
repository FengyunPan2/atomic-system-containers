FROM docker.io/fengyunpan/kubernetes-node:v1.9.6.1
ENV container=docker

ENV NAME=kubernetes-proxy VERSION=0 RELEASE=8 ARCH=x86_64
LABEL bzcomponent="$NAME" \
        name="$FGC/$NAME" \
        version="$VERSION" \
        release="$RELEASE.$DISTTAG" \
        architecture="$ARCH" \
        atomic.type='system' \
        maintainer="FengyunPan <pan_feng_yun@163.com>"

RUN dnf install -y --setopt=tsflags=nodocs iptables conntrack-tools && dnf clean all

LABEL RUN /usr/bin/docker run -d --privileged --net=host 

COPY launch.sh /usr/bin/kube-proxy-docker.sh

COPY service.template config.json.template /exports/

RUN mkdir -p /exports/hostfs/etc/kubernetes && cp /etc/kubernetes/{config,proxy} /exports/hostfs/etc/kubernetes

ENTRYPOINT ["/usr/bin/kube-proxy-docker.sh"]

