ATLAS XCache Docker Image ![Build XCache images from OSG Yum repositories](https://github.com/opensciencegrid/docker-xcache/workflows/Build%20XCache%20images%20from%20OSG%20Yum%20repositories/badge.svg)
=========================

XCache provides a caching service for data federations that serve one or more Virtual Organizations (VOs) based on the
[XRootD](http://xrootd.org/) software.
This image, based on the [OSG XCache image](https://hub.docker.com/r/opensciencegrid/xcache), implements XCache for the
[ATLAS](http://atlas.cern/) data federation.

This document covers how to configure and start an ATLAS XCache container.

Configuration
-------------

Before starting the container, write a file containing the following required environment variables and values for your
XCache:

- `XC_RESOURCENAME`: The server name used for monitoring and reporting
- `XC_SPACE_HIGH_WM`: High watermark for disk usage;
  when usage goes above the high watermark, the cache deletes until it hits the low watermark
- `XC_SPACE_LOW_WM`: Low watermark for disk usage;
  when usage goes above the high watermark, the cache deletes until it hits the low watermark
- `XC_PORT`: TCP port that XCache listens on
- `XC_RAMSIZE`: Amount of memory to use for blocks in flight
- `XC_BLOCKSIZE`: The size of the blocks in the cache
- `XC_PREFETCH`: Number of blocks to prefetch from a file at once

### Use a local redirector (optional) ###

To configure your cache in clustered mode to make use of a local cache redirector, set the `XC_LOCAL_REDIRECTORS` in the
same file as above to a comma-delimited list of redirectors specified by `<FQDN>:<PORT>`.
For example:

```
XC_LOCAL_REDIRECTORS=redir1.example.edu:1094,redir2.example.edu:1094
```


### Disabling OSG monitoring (optional) ###

By default, XCache reports to the OSG so that OSG staff can monitor the health of data federations.
If you would like to report monitoring information to another destination, you can disable the OSG monitoring by setting
the following in your environment variable configuration:

```
DISABLE_OSG_MONITORING = true
```

Running a Container
-------------------

To run the container, use `docker run` with the following options, replacing the text within angle brackets with your
own values:


```
$ docker run --env-file=<PATH TO ENV FILE> \
             --volume <PATH TO HOST CERT>:/etc/grid-security/hostcert.pem \
             --volume <PATH TO HOST KEY>:/etc/grid-security/hostkey.pem \
             --volume <HOST PATH TO CACHE DISK 1>:<CONTAINER MOUNT POINT 1> \
             ...
             --volume <HOST PATH TO CACHE DISK N>:<CONTAINER MOUNT POINT N> \
             --publish <HOST PORT>:<XC_PORT> \
             --hostname <XCACHE HOSTNAME> \
             opensciencegrid/atlas-xcache:development
```
