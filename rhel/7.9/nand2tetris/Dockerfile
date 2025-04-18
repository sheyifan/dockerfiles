FROM registry.redhat.io/rhel7:7.9-189

LABEL com.advantest.yvanshe "Yvan She <yvan.she@ptn.advantest.com>"
LABEL description "Basic desktop environment for Redhat Enterprise Linux server"

# Setup CentOS 7.9 repository
RUN echo -e '[Alibaba_CentOS_7.9.2009]\n\
name=Alibaba CentOS 7.9.2009 mirror\n\
baseurl=https://mirrors.aliyun.com/centos/7.9.2009/os/x86_64/\n\
enabled=true\n\
gpgcheck=true\n\
gpgkey=https://mirrors.aliyun.com/centos/7.9.2009/os/x86_64/RPM-GPG-KEY-CentOS-7' > /etc/yum.repos.d/centos.repo && yum clean all

# Basic softwares
RUN yum install -y tightvnc-server net-tools vim @KDE-desktop openssh-server

ARG OPENJDK_URL=https://github.com/sheyifan/dockerfiles/releases/download/7.9/OpenJDK21U-jdk_x64_linux_hotspot_21.0.6_7.tar.gz

ARG HOME=/root
ARG SW_PATH=/root/workdir/softwares

ARG JAVA_HOME=/root/workdir/sdk/java/jdk-21.0.6+7
ARG MAVEN_HOME=$SW_PATH/idea-IC-2023.1.5/plugins/maven/lib/maven3
ARG MODELSIM_HOME=/root/workdir/softwares/intelFPGA/20.1

RUN mkdir -p $SW_PATH

# Applications
RUN yum install -y wget \
    # Fix chinese messy code
    && yum groupinstall -y "fonts" \
    && mkdir -p /root/workdir/sdk/java/ \
    && wget $OPENJDK_URL -P /root/Downloads \
    && tar xzvf /root/Downloads/OpenJDK21U-jdk_x64_linux_hotspot_21.0.6_7.tar.gz -C /root/workdir/sdk/java/

ARG FONT_HOME=$HOME/.fonts
ARG UBUNTU_FONT_URL=https://assets.ubuntu.com/v1/0cef8205-ubuntu-font-family-0.83.zip

# Customized fonts
RUN yum install -y zip unzip \
    && mkdir -p $FONT_HOME && cd $FONT_HOME \
    # Ubuntu font as default
    && wget --no-check-certificate $UBUNTU_FONT_URL \
    && unzip 0cef8205-ubuntu-font-family-0.83.zip "ubuntu-font-family-0.83/*" -d $FONT_HOME \
    # Install fonts
    && fc-cache -v && fc-cache-64 -v \
    # Jetbrains Mono font
    && /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/JetBrains/JetBrainsMono/master/install_manual.sh)"

RUN yum install -y python3-devel
ARG PYTHON=/usr/bin/python3
# import sysconfig; print(sysconfig.get_config_var('LIBDIR'))
ARG PYTHON_LIBDIR=/usr/lib64

RUN yum groupinstall -y "Development tools" \
    && cd $SW_PATH \
    && yum install -y gcc-gnat \
    # Install GCC
    && wget --no-check-certificate https://ftp.gnu.org/gnu/gcc/gcc-8.1.0/gcc-8.1.0.tar.gz \
    && tar xzvf gcc-8.1.0.tar.gz \
    && cd gcc-8.1.0 \
    && ./contrib/download_prerequisites \
    && ./configure -disable-multilib --enable-languages=c,c++,ada \
    && make -j8 \
    # install gcc to /usr/local/bin/gcc
    && make install \
    && cd ../ && rm -rf gcc-8.1.0.tar.gz \
    # Install GDB
    && wget --no-check-certificate https://ftp.gnu.org/gnu/gdb/gdb-8.1.tar.gz \
    && tar -xzvf gdb-8.1.tar.gz \
    && cd gdb-8.1 \
    # GDB in Qt Creator need to be used with python
    && ./configure --with-python=$PYTHON LDFLAGS="-L$PYTHON_LIBDIR"  \
    && make -j8 \
    && yum install -y texi2html texinfo \
    # install gcc to /usr/local/bin/gdb
    && make install \
    && rm -rf gdb-8.1.tar.gz \
    && yum groupremove -y "Development tools" \
    # Install OpenGL libraries and headers, fix build error in qt creator
    && yum install -y freeglut-devel

# git
RUN yum install -y https://mirrors.aliyun.com/epel/7/x86_64/Packages/e/epel-release-7-14.noarch.rpm \
    && yum install -y https://repo.ius.io/ius-release-el7.rpm \
    && yum install -y make \
    && yum install -y https://repo.ius.io/7/x86_64/packages/g/git236-2.36.5-1.el7.ius.x86_64.rpm

# Python 3.8.5
RUN cd /root/workdir \
    && yum install -y gcc \
    && yum install -y zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel expat-devel libffi-devel \
    && curl https://www.python.org/ftp/python/3.8.5/Python-3.8.5.tgz | tar xz \
    && cd Python-3.8.5 \
    && ./configure --enable-optimizations --enable-shared --with-lto \
    && make -s -j8 \
    && make altinstall \
    && echo "/usr/local/lib" > /etc/ld.so.conf.d/common.conf \
    && ldconfig \
    && cd ../ && rm -rf Python-3.8.5

# Visual Studio Code
RUN rpm --import https://packages.microsoft.com/keys/microsoft.asc \
    && sh -c 'echo -e "[code]\nname=Visual Studio Code\nbaseurl=https://packages.microsoft.com/yumrepos/vscode\nenabled=1\ngpgcheck=1\ngpgkey=https://packages.microsoft.com/keys/microsoft.asc" > /etc/yum.repos.d/vscode.repo' \
    && yum clean all \
    # glibc may not be compatible in higher version
    && yum install -y code-1.80.0 \
    # fix: can not launch vscode as root
    && sed -i '/^Exec=\/usr\/share\/code\/code/ s/$/ --no-sandbox --user-data-dir \/root\/.vscode\/appdata/' /usr/share/applications/code.desktop

# Modelsim
RUN cd /root/Downloads \
    && wget https://github.com/sheyifan/dockerfiles/releases/download/7.9/ModelSimSetup-20.1.0.711-linux.run \
    && chmod a+x ModelSimSetup-20.1.0.711-linux.run \
    # check https://github.com/sheyifan/dockerfiles/issues/16
    && yum distribution-synchronization -y --skip-broken \
    && yum install -y glibc.i686 libX11.i686 libXext.i686 libXft.i686 libgcc.i686 \
    && ./ModelSimSetup-20.1.0.711-linux.run --mode unattended --installdir $MODELSIM_HOME --unattendedmodeui none --accept_eula 1

ARG ZSH_CUSTOM=/root/.oh-my-zsh/custom

# set zsh as default shell
RUN \
    yum install -y zsh \
    # set zsh default shell
    && chsh -s /bin/zsh \
    # install oh-my-zsh
    && sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" \
    # plugin: zsh-completions
    && git clone --depth=1 https://github.com/zsh-users/zsh-completions ${ZSH_CUSTOM:-${ZSH:-~/.oh-my-zsh}/custom}/plugins/zsh-completions \
    # plugin: zsh-autosuggestions
    && git clone --depth=1 https://github.com/zsh-users/zsh-autosuggestions.git ${ZSH_CUSTOM:-${ZSH:-~/.oh-my-zsh}/custom}/plugins/zsh-autosuggestions \
    # plugin: zsh-syntax-highlighting
    && git clone --depth=1 https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting \
    && sed -i 's/plugins=(git)/plugins=(git zsh-autosuggestions zsh-completions zsh-syntax-highlighting)/' /root/.zshrc \
    && sed -i 's/ZSH_THEME="robbyrussell"/ZSH_THEME="af-magic"/' /root/.zshrc \
    # plugin: Incremental completion on zsh (real-time completion)
    && mkdir $ZSH_CUSTOM/plugins/incr && curl -fsSL https://mimosa-pudica.net/src/incr-0.2.zsh -o $ZSH_CUSTOM/plugins/incr/incr.zsh \
    && echo 'source $ZSH_CUSTOM/plugins/incr/incr.zsh' >> ~/.zshrc

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
    # vncconfig for copy-paste between vnc client and server
    echo -e '/usr/sbin/sshd\nvncserver -SecurityTypes None :1 -geometry 1920x1200' \
    > /root/startup.sh \
    && chmod 777 /root/startup.sh

# Disable kwallet
RUN sed -i "s/^Enabled=true/Enabled=false/" /usr/share/kde-settings/kde-profile/default/share/config/kwalletrc

# Exposed port, just as a comment. Enable port mapping by "docker -p"
EXPOSE 22
EXPOSE 5901
# Setup host display for X forward at 6020
ENV DISPLAY=host.docker.internal:20.0

ENV JAVA_HOME=$JAVA_HOME
ENV MAVEN_HOME=$MAVEN_HOME
ENV CMAKE_HOME=/root/workdir/softwares/cmake-3.27.7-linux-x86_64
ENV MODELSIM_HOME=$MODELSIM_HOME
ENV HOME=$HOME
ENV PATH=$CMAKE_HOME/bin:$JAVA_HOME/bin:$MAVEN_HOME/bin:$MODELSIM_HOME/modelsim_ase/bin:$PATH

ENTRYPOINT /root/startup.sh && /usr/bin/zsh

WORKDIR $HOME
