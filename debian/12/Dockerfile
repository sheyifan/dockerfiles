FROM debian:12

# KDE
ENV DEBIAN_FRONTEND=noninteractive
RUN apt update && \
    apt install -y kde-plasma-desktop

# fix: https://stackoverflow.com/questions/63627955/cant-load-shared-library-libqt5core-so-5
RUN apt install -y binutils \
    && strip --remove-section=.note.ABI-tag /usr/lib/x86_64-linux-gnu/libQt5Core.so.5

# VNC
RUN apt install -y tigervnc-standalone-server tigervnc-xorg-extension dbus-x11

# User settings
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

EXPOSE 5900

RUN echo '\
/usr/bin/Xvnc :0 -geometry 1920x1200 -depth 24 -SecurityTypes none -AlwaysShared > /dev/null 2>&1 &\n\
DISPLAY=:0 /usr/bin/startplasma-x11 > /dev/null 2>&1' > /root/startup.sh

ENTRYPOINT /usr/bin/sh /root/startup.sh ; /usr/bin/bash