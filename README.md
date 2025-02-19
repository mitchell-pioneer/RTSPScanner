# RTSPScanner

[![PyPI](https://img.shields.io/pypi/v/rtspscanner)](https://pypi.org/project/rtspscanner/) &nbsp; [![PyPI - Python Version](https://img.shields.io/pypi/pyversions/rtspscanner)](https://pypi.org/project/rtspscanner/) &nbsp; [![PyPI - License](https://img.shields.io/pypi/l/rtspscanner)](https://pypi.org/project/rtspscanner/) [![PyPI - Downloads](https://img.shields.io/pypi/dm/portscanner)](https://pypi.org/project/rtspscanner/)

## Python 3 Utility to scan for RTSP Sources on a network.
This can be used as a command line utility or as a class

This uses a modified version of PortScan forked from [Aperocky/PortScan](https://github.com/Aperocky/PortScan).
A [Pull Request](https://github.com/Aperocky/PortScan/pull/1) has been submitted, but there has not been a commit since 2019.

## Changes 
removed 

https://stackoverflow.com/questions/17309288/importerror-no-module-named-requests


## Requirements
    - Pillow ~= 9.2.0
    - beard-portscan ~= 0.1.5
    - ffmpeg in your $PATH

## Command Line Utility Usage
```
usage: rtspscanner.py [-h] [-w WHITESPACE] [-a ADDRESS] [-n NAME] [-p PORTS] [-pp PATHS] [-c CREDS] -m MODE [-A APIADDR] [-P APIPORT] [-t APITRANSPORT] [-T TIMEOUT]
                      [-R TIMEOUTRETRIES] [-v]

Scans given ports of an IPv4 Address or an IPv4 Network for RTSP streams and adds them to rtsp-simple-server

options:
  -h, --help            show this help message and exit
  -w WHITESPACE, --whitespace WHITESPACE
                        Whitespace Replacement can be - _ or #
  -a ADDRESS, --address ADDRESS
                        Single ipv4 address or ipv4 network in CIDR notation ex: 192.168.0.100 or 192.168.0/24
  -n NAME, --name NAME  Camera Name | only used if single address given
  -p PORTS, --ports PORTS
                        csv format: 554,8554
  -pp PATHS, --paths PATHS
                        csv format: '/Streaming/Channels/101,/live,/live2'
  -c CREDS, --creds CREDS
                        csv formatted user:password pairs: username:password,user:pass
  -m MODE, --mode MODE  add - add cameras found / rem - remove cameras found
  -A APIADDR, --apiaddr APIADDR
                        rtsp-simple-server API IP Address/FQDN
  -P APIPORT, --apiport APIPORT
                        rtsp-simple-server API Port
  -t APITRANSPORT, --apitransport APITRANSPORT
                        rtsp-simple-server API transport (http/https)
  -T TIMEOUT, --timeout TIMEOUT
                        Timeout for ffmpeg command to determine if rtsp stream exists
  -R TIMEOUTRETRIES, --timeoutretries TIMEOUTRETRIES
                        Number of retries on timeout for ffmpeg command to determine if rtsp stream exists
  -v, --verbose         Set verbosity to true
```
## Scanning a Subnet
The utility will scan a given network with CIDR notation
```
$~> rtspscanner -m scan -c admin:admin -a 192.168.2.0/24

4 Potential RTSP Sources:
  192.168.2.63:8554
  192.168.2.240:8554
  192.168.2.190:8554
  192.168.2.189:8554

1 Camera Found:
  192.168.2.190: rtsp://admin:admin@192.168.2.190:8554/Streaming/Channels/101

1 Flaky Camera:
Potential camera that cannot be verfied within 10 second timeout.
This can be increased using the command line option -t <seconds>
  192.168.2.189: rtsp://admin:admin@192.168.2.189:8554/live

Credentials Used:
  admin:admin

Paths Used:
  /Streaming/Channels/101
  /live
```

## Scan Single IP

```
$~> rtspscanner -m scan -c admin:admin -a 192.168.2.189

1 Potential RTSP Source:
  192.168.2.190:8554

1 Camera(s) Found:
  192.168.2.190: rtsp://admin:admin@192.168.2.190:8554/Streaming/Channels/101

Credentials Used:
  admin:admin

Paths Used:
  /Streaming/Channels/101
  /live
```

## Add All Discovered RTSP Cameras to rtsp-simple-server

```
$~> rtspscanner -m add -A 192.168.2.240 -c admin:admin -a 192.168.2.0/24
Adding 192.168.2.190 - rtsp://admin:admin@192.168.2.190:8554/Streaming/Channels/101 | 200 : SUCCESS
Adding 192.168.2.189 - rtsp://admin:admin@192.168.2.189:8554/Streaming/Channels/101 | 200 : SUCCESS

4 Potential RTSP Sources:
  192.168.2.63:8554
  192.168.2.240:8554
  192.168.2.190:8554
  192.168.2.189:8554

2 Cameras Found:
  192.168.2.190: rtsp://admin:admin@192.168.2.190:8554/Streaming/Channels/101
  192.168.2.189: rtsp://admin:admin@192.168.2.189:8554/Streaming/Channels/101

Credentials Used:
  admin:admin

Paths Used:
  /Streaming/Channels/101
  /live
```

## Add Single RTSP Camera to rtsp-simple-server

```
$~> rtspscanner -m add -A 192.168.2.240 -c admin:admin -a 192.168.2.190
Adding 192.168.2.190 - rtsp://admin:admin@192.168.2.190:8554/Streaming/Channels/101 | 200 : SUCCESS

1 Potential RTSP Source:
  192.168.2.190:8554

1 Camera Found:
  192.168.2.190: rtsp://admin:admin@192.168.2.190:8554/Streaming/Channels/101

Credentials Used:
  admin:admin

Paths Used:
  /Streaming/Channels/101
  /live
```

## Use it in a python3 project as a class:
```
from rtspscanner import RTSPScanner
scanner = RTSPScanner()

# Set address to net wth CIDR notation or single host
scanner.address = 192.168.2.0/24

# Set to scan mode (add/rem to/from rtsp-simple-server still under development and considered alpha)
scanner.mode = "scan"

# override default paths in csv (default shown)
# scanner.paths = "live,Streaming/Channels/101"

# override default credentials in csv (default is of type None)
# scanner.creds = "admin:admin,"user:pass"

# override default ports in csv (default shown)
# scanner.ports = "554,8554"

#run the scan
scanner.scan()

# Print the list of cameras found as a dict
print(scanner.scanResults)
```

## Environment variables
Environment variables can be set for all options (useful for use inside a docker container)
```
RTSP_SCAN_PORTS="554,8554"
FFMPEG_TIMEOUT=10
FFMPEG_RETRIES=2
RTSP_SS_ADDRESS="192.168.2.240"
RTSP_SS_PORT=9997
RTSP_SS_TRANSPORT="http"
RTSP_MODE="scan"
RTSP_VERBOSE="false"
RTSP_WHITESPACE="-"
RTSP_CREDS="admin:admin,user:password"
RTSP_PATHS="/Streaming/Channels/101,/live,live2"
RTSP_ADDRESS="192.168.2.0/24"
```