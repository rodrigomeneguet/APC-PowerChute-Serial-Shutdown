FROM opensuse/leap as BUILD

ENV LC_ALL=en_US.utf8 \
    LANG=en_US.utf8 \
    LANGUAGE=en_US.utf8 \
    PCSSDIR=/opt/APC/PowerChuteSerialShutdown/Agent \
    PCSSVER=1.2.0-301

WORKDIR /tmp/pcss/$PCSSVER
RUN set -x \
  && zypper --non-interactive refresh \
  && zypper --non-interactive install tar gzip unzip \
  && curl -o pcssagent-$PCSSVER-EN.x86_64.tar.gz "https://download.schneider-electric.com/files?p_Doc_Ref=SPD-PCSS_LNX_EN&p_enDocType=Software+-+Release&p_File_Name=pcssagent-$PCSSVER-EN.x86_64.tar.gz" \
  && tar -xzf pcssagent-$PCSSVER-EN.x86_64.tar.gz \
  && mkdir -p /etc/rc.d/init.d/ \
  && rpm -ivh --prefix=$PCSSDIR pcssagent-*.x86_64.rpm \
  && cd $PCSSDIR \
  && rm -rf /tmp/pcss/$PCSSVER \
  && ( ../jre/bin/java --add-exports java.base/com.sun.crypto.provider=ALL-UNNAMED -jar ./lib/pcssconfig.jar user=apcadm pass=Qq111111 signal=serial port=/dev/ttyS0 || true )

# ----------

FROM ubuntu:latest

RUN set -x \
  && apt-get update && apt-get install openssh-client -y

ENV PCSSDIR=/opt/APC/PowerChuteSerialShutdown/Agent
ENV JAVA_RUNTIME=$PCSSDIR/../jre/bin/java \
    LD_LIBRARY_PATH=$PCSSDIR/lib/linux64

COPY --from=BUILD /opt/APC/PowerChuteSerialShutdown/ /opt/APC/PowerChuteSerialShutdown/
COPY entrypoint.sh /

WORKDIR $PCSSDIR

RUN set -x \
  && chmod +x /entrypoint.sh \
  && mkdir -p /var/lock/subsys \
  && touch /var/lock/subsys/PBEAgent

EXPOSE 6547/tcp

LABEL org.opencontainers.image.description "APC PowerChute Seriel Shutdown as a Docker container"

ENTRYPOINT ["/entrypoint.sh"]
CMD ["../jre/bin/java", "-Dpicard.main.thread=blocking", "-classpath", "lib/*:Resources/*:comp/*", "com.apcc.m11.application.PicardApplication", "@critical.cfg"]
