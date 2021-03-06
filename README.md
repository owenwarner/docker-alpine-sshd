### Alpine Linux SSHD

A lightweight [OpenSSH][openssh] [Docker image][dockerhub_project] built atop [Alpine Linux][alpine_linux]. Available on [GitHub][github_project].

The root password is "root". SSH host keys (RSA, DSA, ECDSA, and ED25519) are auto-generated when the container is started, unless already present.

#### OpenSSL Version Tags
*Use the specific version tag you're intending to use.*

- `8.1`, `8.1`, `8.1` (OpenSSH_8.1, OpenSSL 1.1.1b, [Dockerfile](https://github.com/owenwarner/docker-alpine-sshd/tree/master/versions/8.1/Dockerfile))
- `7.9-r1`, `7.9` (OpenSSH_7.9, OpenSSL 1.1.1b, [Dockerfile](https://github.com/sickp/docker-alpine-sshd/tree/master/versions/7.9-r1/Dockerfile))
- `7.5-r2` (OpenSSH_7.5p1-hpn14v4, LibreSSL 2.6.3, [Dockerfile](https://github.com/sickp/docker-alpine-sshd/tree/master/versions/7.5-r2/Dockerfile))
- `7.5` (OpenSSH_7.5p1-hpn14v4, LibreSSL 2.5.4, [Dockerfile](https://github.com/sickp/docker-alpine-sshd/tree/master/versions/7.5/Dockerfile))
- `7.4` (OpenSSH_7.4p1, LibreSSL 2.4.4, [Dockerfile](https://github.com/sickp/docker-alpine-sshd/tree/master/versions/7.4/Dockerfile))
- `7.2` (OpenSSH_7.2p2-hpn14v4, OpenSSL 1.0.2j  26 Sep 2016, [Dockerfile](https://github.com/sickp/docker-alpine-sshd/tree/master/versions/7.2/Dockerfile))
- `7.1` (Alpine Linux 3.3, OpenSSH_7.1p2-hpn14v4, OpenSSL 1.0.2e 3 Dec 2015, [Dockerfile](https://github.com/sickp/docker-alpine-sshd/tree/master/versions/7.1/Dockerfile))
- `6.8` (Alpine Linux 3.2, OpenSSH_6.8p1-hpn14v4, OpenSSL 1.0.2d 9 Jul 2015, [Dockerfile](https://github.com/sickp/docker-alpine-sshd/tree/master/versions/6.8/Dockerfile))
- `6.7` (Alpine Linux 3.1, OpenSSH_6.7p1-hpn14v4, OpenSSL 1.0.1p 9 Jul 2015, [Dockerfile](https://github.com/sickp/docker-alpine-sshd/tree/master/versions/6.7/Dockerfile))
- `6.4` (Alpine Linux 2.7, OpenSSH_6.4p1-hpn14v1, OpenSSL 1.0.1p 9 Jul 2015, [Dockerfile](https://github.com/sickp/docker-alpine-sshd/tree/master/versions/6.4/Dockerfile))
- `6.2` (Alpine Linux 2.6, OpenSSH_6.2p2, OpenSSL 1.0.1m 19 Mar 2015, [Dockerfile](https://github.com/sickp/docker-alpine-sshd/tree/master/versions/6.2/Dockerfile))


### Basic Usage

```bash
$ docker run --rm --publish=2222:22 sickp/alpine-sshd:7.9-r1 # /entrypoint.sh
ssh-keygen: generating new host keys: RSA DSA ECDSA ED25519
Server listening on 0.0.0.0 port 22.
Server listening on :: port 22.

$ ssh root@localhost -p 2222  # or $(docker-machine ip default)
# The root password is "root".

$ docker ps | grep 2222
cf8097ea881d        sickp/alpine-sshd:7.9-r1   "/entrypoint.sh"    8 seconds ago       Up 4 seconds        0.0.0.0:2222->22/tcp   stoic_ptolemy
$ docker stop cf80
```

Any arguments are passed to `sshd`. For example, to enable debug output:

```bash
$ docker run --rm --publish=2222:22 sickp/alpine-sshd:7.9-r1 -o LogLevel=DEBUG
...
```

#### Version Info

```bash
$ docker run --rm sickp/alpine-sshd:7.9-r1 -v
...
OpenSSH_7.9p1, OpenSSL 1.1.1b  26 Feb 2019
...

$ docker run --rm --entrypoint=cat sickp/alpine-sshd:7.9-r1 /etc/alpine-release
3.9.4
```

### Customize

This image doesn't attempt to be "the one" solution that suits everyone's needs. It's actually pretty useless in the real world. But it is easy to extend via your own `Dockerfile`. See the [examples directory.][examples]

#### Change root password

Change the root password to something more fun, like "password" or "sunshine":

```dockerfile
FROM sickp/alpine-sshd:latest
RUN echo "root:sunshine" | chpasswd
```

#### Use authorized keys

Disable the root password completely, and use your SSH key instead:

```dockerfile
FROM sickp/alpine-sshd:latest
RUN passwd -d root
COPY identity.pub /root/.ssh/authorized_keys
```

#### Create multiple users

Disable root and create individual user accounts:

```dockerfile
FROM sickp/alpine-sshd:latest
ADD https://github.com/sickp.keys /home/sickp/.ssh/authorized_keys
ADD https://github.com/afrojas.keys /home/afrojas/.ssh/authorized_keys
RUN \
  passwd -d root && \
  adduser -D -s /bin/ash sickp && \
  passwd -u sickp && \
  chown -R sickp:sickp /home/sickp && \
  adduser -D -s /bin/ash afrojas && \
  passwd -u afrojas && \
  chown -R afrojas:afrojas /home/afrojas
```

#### Embed SSH host keys

Embed SSH host keys directly in your private image (via `ssh-keygen -A`), so you can treat your containers like cattle.

```dockerfile
FROM sickp/alpine-sshd:latest
ADD https://github.com/sickp.keys /home/sickp/.ssh/authorized_keys
RUN \
  passwd -d root && \
  adduser -D -s /bin/ash sickp && \
  passwd -u sickp && \
  chown -R sickp:sickp /home/sickp && \
  ssh-keygen -A
```

### History

    2019-05-18 Updated to OpenSSH_7.9p1, OpenSSL 1.1.1b (Alpine Linux 3.9.4).
    2018-02-22 Updated to OpenSSH_7.5.p1-hpn14v4, LibreSSL 2.6.3 (Alpine Linux 3.7.0).
    2017-05-31 Updated to OpenSSH_7.5p1, LibreSSL 2.5.4 (Alpine Linux 3.6.0).
    2017-02-01 Updated to OpenSSH_7.4p1, LibreSSL 2.4.4 (Alpine Linux 3.5.0).
    2016-11-16 Updated to Alpine Linux 3.4.4, OpenSSL 1.0.2j.
    2016-09-29 Updated to Alpine Linux 3.4.3, OpenSSL 1.0.2i.
    2016-06-16 Updated to Alpine Linux 3.4.0 (with `search` support for Kubernetes >=1.2.0).
    2016-03-30 Updated to 7.2p2, OpenSSL 1.0.2g.
    2016-02-09 Added support for ALPINE_NO_RESOLVER in Kubernetes version.
    2016-01-28 Added Kubernetes version, Alpine Linux 3.3.1.
    2015-12-25 Updated 7.1 to Alpine Linux 3.3.
    2015-11-16 Initial version.

[alpine_kubernetes]:  https://hub.docker.com/r/janeczku/alpine-kubernetes/
[alpine_linux]:       https://hub.docker.com/_/alpine/
[dockerhub_project]:  https://hub.docker.com/r/sickp/alpine-sshd/
[examples]:           https://github.com/sickp/docker-alpine-sshd/tree/master/examples/
[github_project]:     https://github.com/sickp/docker-alpine-sshd/
[openssh]:            http://www.openssh.com
