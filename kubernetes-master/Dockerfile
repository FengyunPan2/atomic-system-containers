FROM registry.fedoraproject.org/fedora:27

ENV NAME=kubernetes-master VERSION=0 RELEASE=0 ARCH=x86_64
LABEL bzcomponent="$NAME" \
        name="$FGC/$NAME" \
        version="$VERSION" \
        release="$RELEASE.$DISTTAG" \
        architecture="$ARCH" \
        maintainer="FengyunPan <pan_feng_yun@163.com>"

RUN echo "proxy=http://10.21.0.180:1080" >> /etc/dnf/dnf.conf
RUN echo "http_proxy=http://10.21.0.180:1080" >> /etc/dnf/dnf.conf
RUN echo "https_proxy=https://10.21.0.180:1080" >> /etc/dnf/dnf.conf
RUN echo "ftp_proxy=http://10.21.0.180:1080" >> /etc/dnf/dnf.conf
RUN export http_proxy=http://10.21.0.180:1080 && export https_proxy=https://10.21.0.180:1080 && dnf -y --setopt=tsflags=nodocs update && dnf clean all
RUN groupadd -g 994 kube && useradd -u 996 -g 994 kube
RUN export http_proxy=http://10.21.0.180:1080 && export https_proxy=https://10.21.0.180:1080 && dnf install -y /rpms/kubernetes-client-1.9.6.1-1.el7.centos.x86_64.rpm
RUN export http_proxy=http://10.21.0.180:1080 && export https_proxy=https://10.21.0.180:1080 && dnf install -y /rpms/kubernetes-master-1.9.6.1-1.el7.centos.x86_64.rpm findutils && dnf clean all
