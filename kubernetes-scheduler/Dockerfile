FROM docker.io/fengyunpan/kubernetes-master:v1.9.6.1
ENV container=docker

ENV NAME=kubernetes-scheduler VERSION=0.1 RELEASE=8 ARCH=x86_64
LABEL bzcomponent="$NAME" \
        name="$FGC/$NAME" \
        version="$VERSION" \
        release="$RELEASE.$DISTTAG" \
        architecture="$ARCH" \
        atomic.type='system' \
        maintainer="FengyunPan <pan_feng_yun@163.com>"

COPY launch.sh /usr/bin/kube-scheduler-docker.sh

LABEL RUN /usr/bin/docker run -d --net=host

COPY service.template config.json.template /exports/

RUN mkdir -p /exports/hostfs/etc/kubernetes && cp /etc/kubernetes/{config,scheduler} /exports/hostfs/etc/kubernetes

ENTRYPOINT ["/usr/bin/kube-scheduler-docker.sh"]
