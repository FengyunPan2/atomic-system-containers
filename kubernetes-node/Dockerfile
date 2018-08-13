FROM registry.fedoraproject.org/fedora:26

ENV NAME=kubernetes-node VERSION=0 RELEASE=0 ARCH=x86_64
LABEL bzcomponent="$NAME" \
        name="$FGC/$NAME" \
        version="$VERSION" \
        release="$RELEASE.$DISTTAG" \
        architecture="$ARCH" \
        maintainer="FengyunPan <pan_feng_yun@163.com>"

RUN echo "proxy=http://10.21.0.180:1080" >> /etc/dnf/dnf.conf && echo "http_proxy=http://10.21.0.180:1080" >> /etc/dnf/dnf.conf && echo "https_proxy=https://10.21.0.180:1080" >> /etc/dnf/dnf.conf && echo "ftp_proxy=http://10.21.0.180:1080" >> /etc/dnf/dnf.conf && export http_proxy=http://10.21.0.180:1080 && export https_proxy=https://10.21.0.180:1080 && dnf -y --setopt=tsflags=nodocs update && dnf clean all && dnf -y install dnf-plugins-core && dnf config-manager --add-repo  https://download.docker.com/linux/fedora/docker-ce.repo && dnf config-manager --set-enabled docker-ce-edge && dnf config-manager --set-enabled docker-ce-test && dnf install -y docker-ce  && dnf install -y /rpms/kubernetes-client-1.9.6.1-1.el7.centos.x86_64.rpm && dnf install -y --setopt=tsflags=nodocs /rpms/kubernetes-node-1.9.6.1-1.el7.centos.x86_64.rpm findutils && dnf clean all
