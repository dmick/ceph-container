# CEPH DAEMON IMAGE
# CEPH VERSION: Luminous
# CEPH VERSION DETAIL: 12.x.x

FROM centos:7
MAINTAINER Sébastien Han "seb@redhat.com"

ENV CEPH_VERSION luminous
ENV CONFD_VERSION 0.10.0

# Install Ceph
RUN rpm --import 'https://download.ceph.com/keys/release.asc'
RUN rpm -Uvh http://download.ceph.com/rpm-${CEPH_VERSION}/el7/noarch/ceph-release-1-1.el7.noarch.rpm
RUN yum install -y epel-release
ADD ganesha.repo /etc/yum.repos.d/
RUN yum clean all
ARG PACKAGES="unzip ceph-mon ceph-osd ceph-mds ceph-mgr ceph-base ceph-common ceph-radosgw rbd-mirror device-mapper sharutils etcd kubernetes-client e2fsprogs s3cmd wget nfs-ganesha nfs-ganesha-ceph nfs-ganesha-rgw"
ARG PURGES="/tmp/* /var/tmp/* /usr/lib/{dracut,locale,systemd,udev} /usr/bin/hyperkube /usr/bin/etcd /usr/bin/systemd-analyze /etc/{udev,selinux} /usr/lib/{udev,systemd}"
RUN yum install -y $PACKAGES && rpm -q $PACKAGES && yum clean all && rm -rf $PURGES

# Install confd
ADD https://github.com/kelseyhightower/confd/releases/download/v${CONFD_VERSION}/confd-${CONFD_VERSION}-linux-amd64 /usr/local/bin/confd
RUN chmod +x /usr/local/bin/confd && mkdir -p /etc/confd/conf.d && mkdir -p /etc/confd/templates

# Download forego
RUN wget -O /forego.tgz 'https://bin.equinox.io/c/ekMN3bCZFUn/forego-stable-linux-amd64.tgz' && \
  cd /usr/local/bin && tar xfz /forego.tgz && chmod +x /usr/local/bin/forego && rm /forego.tgz

# Add s3cfg file
ADD s3cfg /root/.s3cfg

# Add bootstrap script, ceph defaults key/values for KV store
ADD *.sh ceph.defaults check_zombie_mons.py ./osd_scenarios/* entrypoint.sh.in disabled_scenario /

# Modify the entrypoint
RUN bash "/generate_entrypoint.sh" && \
  rm -f /generate_entrypoint.sh && \
  bash -n /*.sh

# Add templates for confd
ADD ./confd/templates/* /etc/confd/templates/
ADD ./confd/conf.d/* /etc/confd/conf.d/

# Add volumes for Ceph config and data
VOLUME ["/etc/ceph","/var/lib/ceph", "/etc/ganesha"]

# Execute the entrypoint
WORKDIR /
ENTRYPOINT ["/entrypoint.sh"]
