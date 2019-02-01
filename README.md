# flowexport

Alpine Docker container to expose network flow data.
Based on pmacctd and nfdump.

An automated build is available at Docker Hub:
https://hub.docker.com/r/cha87de/flowexport/

## Usage

Collect flows and export as text to files in folder `/opt/flowexport/nfdump`:

```
docker run -d -ti \
    -e INTERFACE=enp3s0 \
    -e INTERVAL=60 \
    -e MAXAGE=2 \
    -e MODE=text \
    --network=host \
    -v /tmp/nfdump:/opt/flowexport/nfdump \
    cha87de/flowexport:latest
```

Collect flows and send netflow v9 via UDP to `target`:

```
docker run -d -ti \
    -e INTERFACE=enp3s0 \
    -e INTERVAL=60 \
    -e MAXAGE=2 \
    -e MODE=netflow \
    -e TARGET=
    --network=host \
    -v /tmp/nfdump:/opt/flowexport/nfdump \
    cha87de/flowexport:latest
```

Besides single interface, a wildcard interface is supported:

```
docker run -d -ti \
    -e INTERFACE=brq* \
    -e INTERVAL=60 \
    -e MAXAGE=2 \
    --network=host \
    -v /tmp/nfdump:/opt/flowexport/nfdump \
    cha87de/flowexport:latest
```

Environment variables:

 - `INTERFACE`: (required) the interface which will be listened in promiscuous mode. If * at end, one pmacctd per matching interface will be started. Matching interfaces are checked every $INTERVAL seconds.
 - `INTERVAL`: (required) seconds to wait before dumping flows to text files
 - `MODE`: text (default) or netflow
 - `MAXAGE`: (required for `mode` 'text') days after dumps are removed (relevant for file dump mode only)
 - `TARGET`: (required for `mode` 'netflow') host and port to send netflow datagrams to

In MODE=text, the flows will be dumped every $INTERVAL seconds to /opt/flowexport/nfdump as text files, e.g.:

```
-rw-r--r--. 1 root root 1.2K May 15 10:01 201805150759
-rw-r--r--. 1 root root  966 May 15 10:02 201805150800
-rw-r--r--. 1 root root  854 May 15 10:03 201805150801
```

Each file contains the nfdump export for the $INTERVAL timespan and looks like, e.g.:

```
Date first seen          Duration Proto      Src IP Addr:Port          Dst IP Addr:Port   Packets    Bytes Flows
2018-05-15 07:01:28.488     0.000 TCP        123.123.123.123:51481 ->    123.123.123.123:9960         1       46     1
[...]
Summary: total flows: 4, total bytes: 624, total packets: 5, avg bps: 1, avg pps: 0, avg bpp: 124
Time window: 2018-05-15 07:01:17 - 2018-05-15 07:56:29
Total flows processed: 4, Blocks skipped: 0, Bytes read: 344
Sys: 0.006s flows/second: 574.1      Wall: 0.003s flows/second: 1261.0
```
