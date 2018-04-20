# System Containers

As part of our effort to reduce the number of packages that are shipped with
the Atomic Host image, we faced the problem of how to containerize services
that are needed to be run before a container runtime, like the upstream docker
daemon, is running. The result: *system containers*: a way to run containers
in production using read only images.

A system container is a container that is executed out of an systemd unit file
early in boot, using runc. The specified IMAGE must be a system image
already fetched. If it is not already present, atomic will attempt to fetch it
assuming it is an oci image. Installing a system container consists of
checking it the image by default under /var/lib/containers/atomic/ and
generating the configuration files for runc and systemd. OSTree and runc are
required for this feature to be available.

System containers use different technologies:

 * We use the [atomic](https://github.com/projectatomic/atomic) tool to install
 system containers.
 * [Labels](LABELS.md) can influence how the *atomic tool* uses a system container
 * Specific [files](FILES.md) are required to be part of a valid system image
 * For storage system containers do not need to use COW File systems, since
 they are in production. We default to using OSTree for storage of the
 container images.
 * The *atomic tool* does not use upstream docker to pull the container images,
 instead we use the [Skopeo](https://github.com/projectatomic/skopeo) tool to pull images from a container registry.
 * When you *atomic install* a system container the tool will look for a systemd unit file template in /exports directory and will create a systemd unit file to run the container on the host.
 * The unit files uses [runc](https://github.com/opencontainers/runc) to create and run the containers.
 * [systemd](https://github.com/systemd/systemd) manages the lifecycle of the container.

To use system containers you must have Atomic CLI version 1.12 or later and the
ostree utility installed.

For more information on system containers see:

- [Basic usage](USAGE.md)
- http://www.projectatomic.io/blog/2016/09/intro-to-system-containers
- http://www.projectatomic.io/blog/2017/06/creating-system-containers/
- http://www.projectatomic.io/blog/2017/09/running-kubernetes-on-fedora-atomic-26/


1.安装依赖包
# yum install epel-release -y
# yum update -y && yum upgrade -y
# yum install golang go-md2man go-bindata gcc bison git rpm-build vim -y
# yum groupinstall "Development Tools" -y

2.安装gvm
# bash < <(curl -s -S -L https://raw.githubusercontent.com/moovweb/gvm/master/binscripts/gvm-installer)
# source /root/.gvm/scripts/gvm

3.安装golang1.9.2
安装1.9.2 之前需要先安装 1.4
# gvm install go1.4 -B
# gvm use go1.4
使用 golang 1.9.2 版本 build
# gvm install go1.9.2
# gvm use go1.9.2

4.克隆最新的kubernetes build 仓库
# git clone https://src.fedoraproject.org/git/rpms/kubernetes.git
克隆好 build 仓库后首先修改 kubernetes.spec 文件：
# cd kubernetes/
# vim kubernetes.spec 
-----
# git diff
diff --git a/kubernetes.spec b/kubernetes.spec
index 517e843..f48433d 100644
--- a/kubernetes.spec
+++ b/kubernetes.spec
@@ -23,7 +23,7 @@
 
 %global provider_prefix         %{provider}.%{provider_tld}/%{project}/%{repo}
 %global import_path             k8s.io/kubernetes
-%global commit                  9f8ebd171479bec0ada837d7ee641dec2f8c6dd1
+%global commit                  a24e4610c57f69125ff7c07c8ed4087b0876a5b0
 %global shortcommit              %(c=%{commit}; echo ${c:0:7})
 
 %global con_provider            github
@@ -35,7 +35,7 @@
 %global con_commit              5b445f1c53aa8d6457523526340077935f62e691
 %global con_shortcommit         %(c=%{con_commit}; echo ${c:0:7})
 
-%global kube_version            1.9.6
+%global kube_version            1.9.6.1
 %global kube_git_version        v%{kube_version}
 
 # Needed otherwise "version_ldflags=$(kube::version_ldflags)" doesn't work
@@ -805,12 +805,7 @@ Kubernetes services for master host
 %package node
 Summary: Kubernetes services for node host
 
-%if 0%{?fedora} >= 27
-Requires: (docker or docker-ce)
-Suggests: docker
-%else
-Requires: docker
-%endif
+Requires: docker-ce
 Requires: conntrack-tools
 
 BuildRequires: golang >= 1.2-7
注：
1. a24e4610c57f69125ff7c07c8ed4087b0876a5b0是kubernets最后一个commit的uuid
2. 5b445f1c53aa8d6457523526340077935f62e691是contrib最后一个commit的uuid
3. 1.9.6.1是自定义的版本
4. 指定docker-ce

5.下载kubernetes和contrib代码
# wget https://github.com/kubernetes/contrib/archive/5b445f1c53aa8d6457523526340077935f62e691/contrib-5b445f1.tar.gz
# wget https://github.com/kubernetes/kubernetes/archive/a24e4610c57f69125ff7c07c8ed4087b0876a5b0/kubernetes-a24e461.tar.gz

6.创建build目录
# mkdir -p /root/rpmbuild/SOURCES/
# cp * /root/rpmbuild/SOURCES/
# cd /root/rpmbuild/SOURCES/

7.制作kubernetes rpm包
# rpmbuild -bb kubernetes.spec

8.制作完后查看rpm包
# ll /root/rpmbuild/RPMS/x86_64

9.制作kubernetes-master 和 kubernetes-node 镜像
下载atomic-system-containers.tar.gz并解压
# tar -xvf atomic-system-containers.tar.gz
# cd atomic-system-containers
注：里面的文件是build atomic镜像的dockerfile及其相关文件，部分文件设置了代理，制作人员需要修改成相应的代理

# docker build -v /root/rpmbuild/RPMS/x86_64:/rpms  -t kubernetes-master:pfy ./kubernetes-master/
# docker build -v /root/rpmbuild/RPMS/x86_64:/rpms  -t kubernetes-node:pfy ./kubernetes-node/

注：kubernetes-master和kubernetes-node制作完后，需要将其上传到镜像仓库，可以是私有的也可以是公有的，以下使用docker.io为例子
# docker images | grep pfy
kubernetes- master                                     pfy                 55549b3914f8        About an hour ago   958 MB
# docker tag 55549b3914f8 docker.io/fengyunpan/kubernetes-master:v1.9.6.1
# docker push docker.io/fengyunpan/kubernetes-master:v1.9.6.1

10.制作kubernetes各服务对应的atomic镜像
修改kubernetes-apiserver/Dockerfile的‘FROM’指令:
FROM docker.io/fengyunpan/kubernetes-master:v1.9.6.1
注：docker.io/fengyunpan/kubernetes-master:v1.9.6.1是kubernetes-master镜像所有镜像仓库的位置
# docker build  -t kubernetes-apiserver:pfy ./kubernetes-apiserver/
# docker build  -t kubernetes-controller-manager:pfy ./kubernetes-controller-manager/
# docker build  -t kubernetes-kubelet:pfy ./kubernetes-kubelet/
# docker build  -t kubernetes-proxy:pfy ./kubernetes-proxy/
# docker build  -t kubernetes-scheduler:pfy ./kubernetes-scheduler/
# docker images | grep pfy
kubernetes-proxy                                     pfy                 a68e34732328        About an hour ago   958 MB
kubernetes-kubelet                                   pfy                 20f3a0605cb4        About an hour ago   1 GB
kubernetes-scheduler                                 pfy                 c8ab37a6313e        About an hour ago   904 MB
kubernetes-controller-manager                        pfy                 6937e4d7ffff        About an hour ago   904 MB
kubernetes-apiserver                                 pfy                 8ff2a9d4e634        About an hour ago   1.26 GB
kubernetes-node                                      pfy                 053bed094013        About an hour ago   949 MB
kubernetes-master                                    pfy                 fb18b40d351d        About an hour ago   904 MB

11.上传到私有仓库供magnum使用
# docker images | grep 1.9.6.1
docker.io/fengyunpan/kubernetes-proxy                v1.9.6.1            a68e34732328        About an hour ago   958 MB
docker.io/fengyunpan/kubernetes-kubelet              v1.9.6.1            20f3a0605cb4        About an hour ago   1 GB
docker.io/fengyunpan/kubernetes-scheduler            v1.9.6.1            c8ab37a6313e        About an hour ago   904 MB
docker.io/fengyunpan/kubernetes-controller-manager   v1.9.6.1            6937e4d7ffff        About an hour ago   904 MB
docker.io/fengyunpan/kubernetes-apiserver            v1.9.6.1            8ff2a9d4e634        About an hour ago   1.26 GB
docker.io/fengyunpan/kubernetes-node                 v1.9.6.1            053bed094013        About an hour ago   949 MB
docker.io/fengyunpan/kubernetes-master               v1.9.6.1            fb18b40d351d        About an hour ago   904 MB
