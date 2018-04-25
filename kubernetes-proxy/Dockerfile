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

RUN echo "proxy=http://10.21.0.180:1080" >> /etc/dnf/dnf.conf && echo "http_proxy=http://10.21.0.180:1080" >> /etc/dnf/dnf.conf && echo "https_proxy=https://10.21.0.180:1080" >> /etc/dnf/dnf.conf && echo "ftp_proxy=http://10.21.0.180:1080" >> /etc/dnf/dnf.conf && export http_proxy=http://10.21.0.180:1080 && export https_proxy=https://10.21.0.180:1080 && dnf -y --setopt=tsflags=nodocs update && dnf install -y --setopt=tsflags=nodocs iptables conntrack-tools ipvsadm ipset kmod kernel && dnf clean all

RUN modprobe ip_vs && modprobe ip_vs_rr  && modprobe ip_vs_wrr && modprobe ip_vs_sh && lsmod | grep ip_vs

LABEL RUN /usr/bin/docker run -d --privileged --net=host 

COPY launch.sh /usr/bin/kube-proxy-docker.sh

COPY service.template config.json.template /exports/

RUN mkdir -p /exports/hostfs/etc/kubernetes && cp /etc/kubernetes/{config,proxy} /exports/hostfs/etc/kubernetes

ENTRYPOINT ["/usr/bin/kube-proxy-docker.sh"]

