# 7 Working with Containers

## Containers Big Picture

Container => Atomic Unit

Build Time Container Layers => Read Only
Run Time Container Layer => Writable

**Every Micro Service has is Container** 

## Diving Deeper

exit container: ctrl+p+q

```bash
docker container exec ID sh
```

## Logging

*   Daemon logs: 
    *   Linux  
        systemd: journalctl u docker.service
    *   Windows:
        ~/AppData/Local/Docker
*   STDOUT/STDERR in daemon.json

