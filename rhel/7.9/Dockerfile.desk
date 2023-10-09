FROM registry.redhat.io/rhel7:7.9-189

LABEL com.advantest.yvanshe "Yvan She <yvan.she@ptn.advantest.com>"
LABEL description "Jetbrains IDEA Community Development Kit for Redhat Enterprise Linux server 7.9"

# Setup CentOS 7.9 repository
RUN echo -e '[CentOS_7.9.2009]\nname=CentOS 7.9.2009 mirror\nbaseurl=http://mirror.centos.org/centos/7.9.2009/os/x86_64/\nenabled=1\ngpgcheck=1\ngpgkey=http://mirror.centos.org/centos/7.9.2009/os/x86_64/RPM-GPG-KEY-CentOS-7' \
    > /etc/yum.repos.d/centos.repo \
    && yum clean all

# Basic softwares
RUN yum install -y tightvnc-server net-tools vim @KDE-desktop openssh-server

# Applications
RUN yum install -y wget \
    && wget --no-check-certificate https://download-installer.cdn.mozilla.net/pub/firefox/releases/117.0/linux-x86_64/en-US/firefox-117.0.tar.bz2 -P /root/Downloads \
    # Download firefox
    && mkdir -p /root/workdir/softwares && tar -xjvf /root/Downloads/firefox-117.0.tar.bz2 -C /root/workdir/softwares \
    ## Install firefox
    && mv /root/workdir/softwares/firefox /opt \
    && ln -s /opt/firefox/firefox /usr/local/bin/firefox \
    # Setup firefox desktop application
    && wget https://raw.githubusercontent.com/mozilla/sumo-kb/main/install-firefox-linux/firefox.desktop -P /usr/local/share/applications \
    && rm /root/Downloads/firefox-117.0.tar.bz2 \
    # Ubuntu font, for chinese & English
    && mkdir -p /root/.fonts && cd /root/.fonts && wget https://assets.ubuntu.com/v1/0cef8205-ubuntu-font-family-0.83.zip && unzip 0cef8205-ubuntu-font-family-0.83.zip ubuntu-font-family-0.83/* -d ./ && rm  0cef8205-ubuntu-font-family-0.83.zip \
    && fc-cache -v && fc-cache-64 -v \
    # Jetbrains Mono font for coding
    && /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/JetBrains/JetBrainsMono/master/install_manual.sh)"

# Login settings
RUN \
    sed -i 's/#PermitRootLogin yes/PermitRootLogin yes/' /etc/ssh/sshd_config \
    # Must be setup, or vnc server will not be launched
    && dbus-uuidgen > /etc/machine-id

RUN \
    # Generate Key by RSA, no passphase
    ssh-keygen -t rsa -N "" -f /etc/ssh/ssh_host_rsa_key <<< y \
    && ssh-keygen -t ecdsa -N "" -f /etc/ssh/ssh_host_ecdsa_key <<< y \
    && ssh-keygen -t ed25519 -N "" -f /etc/ssh/ssh_host_ed25519_key <<< y \
    # Enable SSH Login as root
    && sed -i 's/#PermitRootLogin yes/PermitRootLogin yes/' /etc/ssh/sshd_config \
    && mkdir -p /root/.ssh && chmod 700 /root/.ssh \
    # Where to put public key from client
    && touch /root/.ssh/authorized_keys && chmod 640 /root/.ssh/authorized_keys

# Startup script
RUN \
    echo -e '/usr/sbin/sshd\nvncserver -SecurityTypes None :1 -geometry 1920x1200' \
    > /root/startup.sh \
    && chmod 777 /root/startup.sh

# Exposed port, just as a comment. Enable port mapping by "docker -p"
EXPOSE 22
EXPOSE 5901
# Setup host display for X forward at 6020
ENV DISPLAY=host.docker.internal:20.0

ENTRYPOINT /root/startup.sh && /usr/bin/bash