# Dockerized ClamAV Private Mirror
This is a dockerized database mirror for the Clam Antivirus. 

While by default using official mirrors ClamAV allows the configuration of a `PrivateMirror` or `DatabaseMirror` to be a custom hostname or IP address under your management.

The official mirrors are of course the most accurate and probably the fastest way of upding the local database, however they tend to rate limit your IP address after multiple updating processes in some timewindow (through updater tool `FreshClam`). 
This becomes more of a problem when you have more than one ClamAV host with periodic updates.

In this case, it is better to host your own private mirror of the Clam Antivirus Database. 
The private mirror which is this dockerized project, updates its database every couple (configurable) hours and all ClamAV hosts in the local network should be configured to now pull the database from the private mirror instead of the offical ones.

## Build

```
podman build . -t clamav-mirror:v.0.3
```

## Deployment
1. Create persistenet volume `clamav`
2. Create deployment manifest
```
kubectl create -f k8s/deployment.yaml
kubectl create -f k8s/service.yaml

```

## Monitor the cvd updates from verbose log

```
WARNING:cvdupdate-1.0.2:You are running cvdupate version: 1.0.2.
WARNING:cvdupdate-1.0.2:There is a newer version on PyPI: shouldstartwithhttp://orhttps://. Please update!
INFO:cvdupdate-1.0.2:main.cvd is up-to-date. Version: 62
INFO:cvdupdate-1.0.2:daily.cvd is up-to-date. Version: 26642
INFO:cvdupdate-1.0.2:bytecode.cvd is up-to-date. Version: 333
```


In the freshclam.conf on ClamAV host, set: (see example/freshclamd.conf)

```
DatabaseMirror http://clamav-mirror.cra.svc.cluster.local:8080
```


## Monitor the client retrieving the updates
```
192.168.58.32 - - [29/Aug/2022 18:34:33] "GET /daily.cvd HTTP/1.1" 200 -
192.168.58.31 - - [29/Aug/2022 18:35:13] "GET /main.cvd HTTP/1.1" 200 -
192.168.58.31 - - [29/Aug/2022 18:36:13] "GET /bytecode.cvd HTTP/1.1" 200 -
192.168.46.107 - - [20/Sep/2022 14:57:14] "GET /daily-26664.cdiff HTTP/1.1" 200 -
192.168.16.19 - - [20/Sep/2022 15:23:05] "GET /daily-26664.cdiff HTTP/1.1" 200 -
192.168.16.23 - - [20/Sep/2022 15:23:05] "GET /daily-26664.cdiff HTTP/1.1" 200 -
```


## Third Party
For updating the dataase, the Python tool `cvdupdate` developed by Micah Snyder (co-dev of ClamAV) is used.

* [cvdupdate](https://github.com/micahsnyder/cvdupdate/blob/main/LICENSE): This tool downloads the latest ClamAV databases along with the latest database patch files.

## Configuration
So far the only possible configuration is to rebuild the image with changed parameters in `mirror.py`.

Environment variables will soon be available for configuration of the mirror.


