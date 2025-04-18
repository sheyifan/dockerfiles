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

ARG FIREFOX_URL=https://download-installer.cdn.mozilla.net/pub/firefox/releases/117.0/linux-x86_64/en-US/firefox-117.0.tar.bz2
ARG FIREFOX_DESKTOP_URL=https://raw.githubusercontent.com/mozilla/sumo-kb/main/install-firefox-linux/firefox.desktop
ARG OPENJDK_URL=https://d6.injdk.cn/openjdk/openjdk/17/openjdk-17.0.1_linux-x64_bin.tar.gz
ARG IDEA_URL=https://download-cdn.jetbrains.com/idea/ideaIC-2023.1.5.tar.gz

ARG HOME=/root
ARG SW_PATH=/root/workdir/softwares

ARG JAVA_HOME=/root/workdir/sdk/java/17/jdk-17.0.1
ARG MAVEN_HOME=$SW_PATH/idea-IC-2023.1.5/plugins/maven/lib/maven3
ARG IDEA_HOME=$SW_PATH/idea-IC-2023.1.5

RUN mkdir -p /root/.local/share/applications \
&& echo -e '[Desktop Entry]\n\
Version=1.0\n\
Type=Application\n\
Name=IntelliJ IDEA Community Edition\n\
Icon=/root/workdir/softwares/idea-IC-2023.1.5/bin/idea.svg\n\
Exec="/root/workdir/softwares/idea-IC-2023.1.5/bin/idea.sh" %f\n\
Comment=Capable and Ergonomic IDE for JVM\n\ 
Categories=Development;IDE;\n\
Terminal=false\n\
StartupWMClass=jetbrains-idea-ce\n\
StartupNotify=true' > /root/.local/share/applications/jetbrains-idea-ce.desktop

# Applications
RUN yum install -y wget \
    && wget --no-check-certificate $FIREFOX_URL -P /root/Downloads \
    # Download firefox
    && mkdir -p $SW_PATH && tar -xjvf /root/Downloads/firefox-117.0.tar.bz2 -C $SW_PATH \
    ## Install firefox
    && mv $SW_PATH/firefox /opt \
    && ln -s /opt/firefox/firefox /usr/local/bin/firefox \
    # Setup firefox desktop application
    && wget $FIREFOX_DESKTOP_URL -P /usr/local/share/applications \
    && rm /root/Downloads/firefox-117.0.tar.bz2 \
    # Fix chinese messy code
    && yum groupinstall -y "fonts" \
    && mkdir -p /root/workdir/sdk/java/17 \
    && wget $OPENJDK_URL -P /root/Downloads \
    && tar xzvf /root/Downloads/openjdk-17.0.1_linux-x64_bin.tar.gz -C /root/workdir/sdk/java/17 \
    && wget $IDEA_URL -P $SW_PATH && tar xzvf $SW_PATH/ideaIC-2023.1.5.tar.gz -C $SW_PATH && mv $SW_PATH/idea-IC* $IDEA_HOME \
    && rm $SW_PATH/ideaIC-2023.1.5.tar.gz

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

# glibc
RUN yum groupinstall -y "Development tools" \
    cd /root/Downloads \
    && wget -qO - https://mirrors.tuna.tsinghua.edu.cn/gnu/glibc/glibc-2.18.tar.gz | tar -xzvf - \
    && cd glibc-2.18 && mkdir build && cd build \
    && ../configure --prefix=$SW_PATH/glibc/2.18 \
    && make -j8 && make install

RUN cd $SW_PATH \
    && wget https://github.com/Kitware/CMake/releases/download/v3.27.7/cmake-3.27.7-linux-x86_64.tar.gz \
    && tar xzvf cmake-3.27.7-linux-x86_64.tar.gz \
    && wget --no-check-certificate https://upload.wikimedia.org/wikipedia/commons/1/13/Cmake.svg -P $SW_PATH/cmake-3.27.7-linux-x86_64 \
    && rm -rf cmake-3.27.7-linux-x86_64.tar.gz \
    && echo -e '[Desktop Entry]\n\
Version=1.0\n\
Type=Application\n\
Name=CMake\n\
Icon=/root/workdir/softwares/cmake-3.27.7-linux-x86_64/Cmake.svg\n\
Exec="/root/workdir/softwares/cmake-3.27.7-linux-x86_64/bin/cmake-gui"\n\
Comment=CMake GUI\n\ 
Categories=Development;\n\
Terminal=false\n\
StartupWMClass=cmake-gui\n\
StartupNotify=true' > /root/.local/share/applications/cmake.desktop

# Visual Studio Code
RUN rpm --import https://packages.microsoft.com/keys/microsoft.asc \
    && sh -c 'echo -e "[code]\nname=Visual Studio Code\nbaseurl=https://packages.microsoft.com/yumrepos/vscode\nenabled=1\ngpgcheck=1\ngpgkey=https://packages.microsoft.com/keys/microsoft.asc" > /etc/yum.repos.d/vscode.repo' \
    && yum clean all \
    # glibc may not be compatible in higher version
    && yum install -y code-1.80.0

RUN cd $SW_PATH \
    # Install GCC
    && wget --no-check-certificate https://ftp.gnu.org/gnu/gcc/gcc-13.2.0/gcc-13.2.0.tar.gz \
    && tar xzvf gcc-13.2.0.tar.gz \
    && cd gcc-13.2.0 \
    && ./contrib/download_prerequisites \
    && ./configure -disable-multilib \
    && make -j8 \
    # install gcc to /usr/local/bin/gcc
    && make install \
    && cd ../ && rm -rf gcc-13.2.0.tar.gz

RUN yum install -y python3-devel
ARG PYTHON=/usr/bin/python3
# import sysconfig; print(sysconfig.get_config_var('LIBDIR'))
ARG PYTHON_LIBDIR=/usr/lib64

RUN \
    # Fix: GMP is missing or unusable
    yum install -y gmp-devel \
    && cd $SW_PATH \
    # Install GDB
    && wget --no-check-certificate https://ftp.gnu.org/gnu/gdb/gdb-13.2.tar.gz \
    && tar -xzvf gdb-13.2.tar.gz \
    && cd gdb-13.2 \
    # GDB in Qt Creator need to be used with python
    && ./configure --with-python=$PYTHON LDFLAGS="-L$PYTHON_LIBDIR"  \
    && make -j8 \
    && yum install -y texi2html texinfo \
    # install gcc to /usr/local/bin/gdb
    && make install \
    && rm -rf gdb-13.2.tar.gz \
    && yum groupremove -y "Development tools" \
    # Install OpenGL libraries and headers, fix build error in qt creator
    && yum install -y freeglut-devel

# git
RUN yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm https://repo.ius.io/ius-release-el7.rpm \
    && yum install -y make \
    && yum install -y https://repo.ius.io/7/x86_64/packages/g/git236-2.36.5-1.el7.ius.x86_64.rpm

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

# LLVM
RUN \
    cd $SW_PATH \
    && wget https://github.com/clangd/clangd/releases/download/18.1.3/clangd-linux-18.1.3.zip && unzip clangd-linux-18.1.3.zip && rm clangd-linux-18.1.3.zip \
    # flag for clangd to search headers: -isystem /root/workdir/softwares/clangd_18.1.3/lib/clang/18/include/
    && echo -e "/root/workdir/softwares/glibc/2.18/lib/ld-2.18.so /root/workdir/softwares/clangd_18.1.3/bin/clangd \$@" > /usr/local/bin/clangd && chmod a+x /usr/local/bin/clangd \
    # clangd will find header from <launch_dir>/../lib
    && cp -r /root/workdir/softwares/clangd_18.1.3/lib/clang /root/workdir/softwares/glibc/2.18/lib \
    # link to stdc++ bundled with built gcc
    && ln -sf /usr/local/lib64/libstdc++.so.6.0.32 /lib64/libstdc++.so.6 \
    # build llvm
    && cd /root/Downloads \
    && git clone --depth=1 -b llvmorg-18.1.3 --single-branch https://github.com/llvm/llvm-project.git \
    && cd llvm-project && mkdir build && cd build \
    && cmake -DCMAKE_INSTALL_PREFIX:PATH=$SW_PATH/llvm -DLLVM_ENABLE_PROJECTS="clang;clang-tools-extra" -DCMAKE_BUILD_TYPE=Release -G "Unix Makefiles" ../llvm \
    && make -j8 && make install \
    && ln -s $SW_PATH/llvm/bin/clang-format /usr/local/bin/clang-format \
    && ln -s /root/workdir/softwares/llvm/bin/clang-tidy /usr/local/bin/clang-tidy

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
    echo -e '/usr/sbin/sshd\nvncserver -SecurityTypes None :1 -geometry 1920x1200\n' \
    > /root/startup.sh \
    && chmod 777 /root/startup.sh

RUN \
    # vscode cannot be launched by root user without some specific parameters
    echo -e 'code() { command code -parameter "--no-sandbox" "--user-data-dir" "/tmp/vscode" "$@" }'

# Exposed port, just as a comment. Enable port mapping by "docker -p"
EXPOSE 22
EXPOSE 5901
# Setup host display for X forward at 6020
ENV DISPLAY=host.docker.internal:20.0

ENV JAVA_HOME=$JAVA_HOME
ENV MAVEN_HOME=$MAVEN_HOME
ENV IDEA_HOME=$IDEA_HOME
ENV CMAKE_HOME=/root/workdir/softwares/cmake-3.27.7-linux-x86_64
ENV HOME=$HOME
ENV PATH=$CMAKE_HOME/bin:$JAVA_HOME/bin:$MAVEN_HOME/bin:$IDEA_HOME/bin:$PATH

ENTRYPOINT /root/startup.sh && /usr/bin/zsh

WORKDIR $HOME
