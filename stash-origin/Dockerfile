FROM opensciencegrid/xcache:fresh

LABEL maintainer OSG Software <help@opensciencegrid.org>

# Specify the base Yum repository to get the necessary RPMs
ARG BASE_YUM_REPO

ENV XC_IMAGE_NAME stash-origin

# Do not stomp on host volume mount ownership
# Files and dirs should be readable to UID/GID 10940:10940 or the world
ENV XC_FIX_DIR_OWNERS no

RUN yum -y install stash-origin --enablerepo="$BASE_YUM_REPO" && \
    yum clean all --enablerepo=* && rm -rf /var/cache/yum/

ADD cron.d/* /etc/cron.d/
RUN chmod 0644 /etc/cron.d/*
ADD image-config.d/* /etc/osg/image-config.d/
ADD supervisord.d/* /etc/supervisord.d/

ADD xrootd/* /etc/xrootd/config.d/
# Add a placeholder scitokens.conf file, in case this origin isn't registered
# and can't pull down a new one
ADD scitokens.conf /run/stash-origin-auth/scitokens.conf
