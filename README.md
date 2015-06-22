## runc

_NOTE:_ Since this is a big change from the current `opencontainer/runc` project I have left the import paths pointing to `gwoo/runc`

_NOTE:_ Some vendor trickery used because libcontainer includes a `vendor/src` directory, but still references the default import paths for vendored dependencies. For example, `github.com/coreos/go-systemd` is import path, but `libcontainer/vendor/src/github.com/coreos/go-systemd` is the desired dependency. The `libcontainer` project manages this by including all the files on the command line in the Makefile. This project solve the problem by updating the GOPATH and only including dependencies not part of the `libcontainer` project in the Godeps manifest.

`runc` is a CLI tool for spawning and running containers according to the OCF specification.

### Building:

```bash
git clone https://github.com/gwoo/runc
make install
```

### Using:

To run a container that you received just execute `runc run` with the JSON format at the argument or have a
`container.json` file in the current working directory.

```bash
runc
/ $ ps
PID   USER     COMMAND
1     daemon   sh
5     daemon   sh
/ $
```

### OCF Container JSON Format;

```json
{
    "version": "0.1",
    "os": "linux",
    "arch": "amd64",
    "processes": [
        {
            "tty": true,
            "user": "daemon",
            "args": [
                "sh"
            ],
            "env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "TERM=xterm"
            ],
            "cwd": ""
        }
    ],
    "root": {
        "path": "rootfs",
        "readonly": true
    },
    "cpus": 1.1,
    "memory": 1024,
    "hostname": "shell",
    "namespaces": [
        {
            "type": "process"
        },
        {
            "type": "network"
        },
        {
            "type": "mount"
        },
        {
            "type": "ipc"
        },
        {
            "type": "uts"
        }
    ],
    "capabilities": [
        "AUDIT_WRITE",
        "KILL",
        "NET_BIND_SERVICE"
    ],
    "devices": [
        "null",
        "random",
        "full",
        "tty",
        "zero",
        "urandom"
    ],
    "mounts": [
        {
            "type": "proc",
            "source": "proc",
            "destination": "/proc",
            "options": ""
        },
        {
            "type": "tmpfs",
            "source": "tmpfs",
            "destination": "/dev",
            "options": "nosuid,strictatime,mode=755,size=65536k"
        },
        {
            "type": "devpts",
            "source": "devpts",
            "destination": "/dev/pts",
            "options": "nosuid,noexec,newinstance,ptmxmode=0666,mode=0620,gid=5"
        },
        {
            "type": "tmpfs",
            "source": "shm",
            "destination": "/dev/shm",
            "options": "nosuid,noexec,nodev,mode=1777,size=65536k"
        },
        {
            "type": "mqueue",
            "source": "mqueue",
            "destination": "/dev/mqueue",
            "options": "nosuid,noexec,nodev"
        },
        {
            "type": "sysfs",
            "source": "sysfs",
            "destination": "/sys",
            "options": "nosuid,noexec,nodev"
        }
    ]
}
```

### Examples:

#### Using runc with systemd

```service
[Unit]
Description=Minecraft Build Server
Documentation=http://minecraft.net
After=network.target

[Service]
CPUQuota=200%
MemoryLimit=1536M
ExecStart=/usr/local/bin/runc
Restart=on-failure
WorkingDirectory=/containers/minecraftbuild

[Install]
WantedBy=multi-user.target
```
