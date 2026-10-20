ARG BASE_IMAGE=hub.opensciencegrid.org/osg-htc/xcache:25-testing-20261018-0750

# The only xrootd-rucioN2N-for-Xcache RPM (osg-contrib) is built against xrootd 5 and
# requires xrootd < 6, so rebuild it from source against the xrootd 6 in the base image.
FROM ${BASE_IMAGE} AS rucion2n-builder

ARG RUCION2N_COMMIT=f35516271c5f7256714a36d760287a048b55a2e5
ARG RUCION2N_VERSION=1.2

RUN XRD_VR=$(rpm -q --qf '%{VERSION}-%{RELEASE}' xrootd-server) && \
    yum install -y rpm-build gcc-c++ make libcurl-devel openssl-devel libxml2-devel \
        xrootd-devel-$XRD_VR xrootd-server-devel-$XRD_VR xrootd-client-devel-$XRD_VR

RUN mkdir -p /root/rpmbuild/SOURCES /root/rpmbuild/SPECS /tmp/src && cd /tmp/src && \
    curl -fsSL https://github.com/xrootd/rucioN2N-for-Xcache/archive/$RUCION2N_COMMIT.tar.gz | tar xz && \
    mv rucioN2N-for-Xcache-$RUCION2N_COMMIT rucioN2N-for-Xcache-$RUCION2N_VERSION && \
    tar czf /root/rpmbuild/SOURCES/rucioN2N-for-Xcache-$RUCION2N_VERSION.tar.gz rucioN2N-for-Xcache-$RUCION2N_VERSION && \
    sed 's/^Release:.*/Release:   99.xrd6%{?dist}/' rucioN2N-for-Xcache-$RUCION2N_VERSION/rucioN2N-for-Xcache.spec \
        > /root/rpmbuild/SPECS/rucioN2N-for-Xcache.spec && \
    rpmbuild -bb --define 'debug_package %{nil}' /root/rpmbuild/SPECS/rucioN2N-for-Xcache.spec


FROM ${BASE_IMAGE}

LABEL maintainer="ivukotic@uchicago.edu"

# Specify the base Yum repository to get the necessary RPMs.
ARG BASE_YUM_REPO=osg-upcoming-testing
ARG BASE_OSG_SERIES=25
ARG BASE_OS=el9

ENV XC_IMAGE_NAME=atlas-xcache

RUN yum update -y --enablerepo="$BASE_YUM_REPO"

COPY --from=rucion2n-builder /root/rpmbuild/RPMS/x86_64/xrootd-rucioN2N-for-Xcache-*.rpm /tmp/rpms/
RUN yum install -y /tmp/rpms/*.rpm && rm -rf /tmp/rpms

# osg-contrib is deliberately not enabled: its xrootd-rucioN2N-for-Xcache would downgrade xrootd to 5.x.
# xrdcl-http is not needed: xrootd 6 ships the HTTP client plugin in xrootd-client-libs.
RUN yum install -y atlas-xcache --enablerepo="$BASE_YUM_REPO" --exclude=alja-\*

# Fail the build if anything pulled xrootd back to 5.x.
RUN rpm -q xrootd-server | grep -q '^xrootd-server-6\.'

RUN yum install -y python3 python3-psutil python3-requests && \
    yum clean all --enablerepo=* && rm -rf /var/cache/



COPY gStream2tcp.py stats.py heartbeat.sh dark-data-cleaner.py /usr/local/sbin/
COPY runme.sh runme_sv.sh run_cache_reporter.sh run_x509_updater.sh /usr/local/sbin/

RUN rm /etc/xrootd/config.d/50-osg-monitoring.cfg /etc/xrootd/config.d/50-osg-http.cfg
RUN rm /etc/xrootd/config.d/*
COPY config.d/* /etc/xrootd/config.d/
RUN chmod -x /etc/xrootd/config.d/*


COPY 10-atlas-xcache-limits.conf /etc/security/limits.d
COPY supervisord.d/10-atlas-xcache.conf /etc/supervisord.d/
COPY image-config.d/10-atlas-xcache.sh /etc/osg/image-config.d/


RUN echo "g /atlas / rl" > /etc/xrootd/auth_db
RUN touch /var/run/x509up

# not sure this line is needed
RUN mkdir -p /xrd/var/log /xrd/var/spool /xrd/var/run /tests

# not sure the two lines bellow are needed at all
# if needed change ownership of directories
RUN if [ $(stat -c "%U:%G" /xrd/var ) != "xrootd:xrootd" ]; then chown -R xrootd:xrootd /xrd/var; fi
RUN if [ $(stat -c "%U:%G" /xrd ) != "xrootd:xrootd" ]; then chown -R xrootd:xrootd /xrd; fi

CMD [ "/usr/local/sbin/runme.sh" ]

