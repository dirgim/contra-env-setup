FROM fedora:28
MAINTAINER "Krunoslav Pavic" <kpavic@redhat.com>

ENV container docker
ENV HOME=/home

WORKDIR $HOME

COPY prepare_and_test.sh $HOME/
COPY test_contra_env_setup.py $HOME/
COPY debug_vars.yml $HOME/

RUN dnf -y install libguestfs libguestfs-tools-c \
                   git libvirt-client \
                   PyYAML python2-setuptools virt-install \
                   python-pip python-devel gcc net-tools \
                   openssh-clients sshpass \
                   libselinux-python libffi-devel\
                   openssl-devel openssh-server libyaml-devel \
                   python-lxml libvirt libvirt-devel \
                   libvirt-python ansible pytest \
                   && dnf clean all

RUN dnf -y update && dnf clean all
RUN dnf -y install systemd

RUN ssh-keygen -f /root/.ssh/id_rsa -t rsa -N ''
RUN printf "Host *\n    StrictHostKeyChecking no" > /root/.ssh/config

RUN dnf -y install libvirt-daemon-driver-* libvirt-daemon libvirt-daemon-kvm qemu-kvm socat && dnf clean all; \
(cd /lib/systemd/system/sysinit.target.wants/; for i in *; do [ $i == systemd-tmpfiles-setup.service ] || rm -f $i; done); \
rm -f /lib/systemd/system/multi-user.target.wants/*;\
rm -f /etc/systemd/system/*.wants/*;\
rm -f /lib/systemd/system/local-fs.target.wants/*; \
rm -f /lib/systemd/system/sockets.target.wants/*udev*; \
rm -f /lib/systemd/system/sockets.target.wants/*initctl*; \
rm -f /lib/systemd/system/basic.target.wants/*;\
rm -f /lib/systemd/system/anaconda.target.wants/*; \
systemctl  enable libvirtd; \
systemctl enable virtlockd: \
systemctl enable debug-shell.service

COPY default.xml /etc/libvirt/qemu/networks/

RUN pip install ara

# Edit the service file which includes ExecStartPost to chmod /dev/kvm
RUN sed -i "/Service/a ExecStartPost=\/bin\/chmod 666 /dev/kvm" /usr/lib/systemd/system/libvirtd.service
RUN echo "namespaces = []" >> /etc/libvirt/qemu.conf

VOLUME ["/sys/fs/cgroup", "/run", "/tmp"]
CMD ["/usr/sbin/init"]