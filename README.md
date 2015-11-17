### Alpine Linux SSHD

A simple, [dockerized, SSH service][alpine_sshd] built on top of [Glider Labs's Alpine Linux][gliderlabs_alpine] Docker image.

The root password is "root".

Auto-generates SSH host keys (RSA, DSA, ECDSA, and ED25519) when the container is started, unless already present.

#### Basic usage

```sh
$ docker run -dP --name=sshd sickp/alpine-sshd
4d9060b18efad357d1216704068006d0dcaadf06367dbd3f551d1e943aabb5ec

$ docker logs sshd
ssh-keygen: generating new host keys: RSA DSA ECDSA ED25519
Server listening on 0.0.0.0 port 22.
Server listening on :: port 22.

$ docker port sshd
22/tcp -> 0.0.0.0:32768

$ ssh root@localhost -p 32768 # on Mac/Windows replace localhost with $(docker-machine ip default)
# The root password is "root".
```

Any additional arguments are passed to `sshd`. For example, to enable debug output:

```sh
$ docker run -dP --name=sshd sickp/alpine-sshd -o LogLevel=DEBUG
```

#### Supported tags and `Dockerfile` links

* [`7.1`][dockerfile_7_1], [`latest`][dockerfile_7_1] (Alpine Linux Edge, OpenSSH_7.1p1-hpn14v4, OpenSSL 1.0.2d 9 Jul 2015)
* [`6.8`][dockerfile_6_8] (Alpine Linux 3.2.x, OpenSSH_6.8p1-hpn14v4, OpenSSL 1.0.2d 9 Jul 2015)
* [`6.7`][dockerfile_6_7] (Alpine Linux 3.1.x, OpenSSH_6.7p1-hpn14v4, OpenSSL 1.0.1p 9 Jul 2015)
* [`6.4`][dockerfile_6_4] (Alpine Linux 2.7.x, OpenSSH_6.4p1-hpn14v1, OpenSSL 1.0.1p 9 Jul 2015)
* [`6.2`][dockerfile_6_2] (Alpine Linux 2.6.x, OpenSSH_6.2p2, OpenSSL 1.0.1m 19 Mar 2015)

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

Embed SSH host keys directly in your private image, so you can treat your containers like cattle.

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

- 2015-11-16 Initial version, based on [Centos SSHD][centos_sshd].

[alpine_sshd]:       https://hub.docker.com/r/sickp/alpine-sshd/
[gliderlabs_alpine]: https://hub.docker.com/r/gliderlabs/alpine/
[dockerfile_7_1]:    https://github.com/sickp/docker-alpine-sshd/tree/master/versions/7.1/Dockerfile
[dockerfile_6_8]:    https://github.com/sickp/docker-alpine-sshd/tree/master/versions/6.8/Dockerfile
[dockerfile_6_7]:    https://github.com/sickp/docker-alpine-sshd/tree/master/versions/6.7/Dockerfile
[dockerfile_6_4]:    https://github.com/sickp/docker-alpine-sshd/tree/master/versions/6.4/Dockerfile
[dockerfile_6_2]:    https://github.com/sickp/docker-alpine-sshd/tree/master/versions/6.2/Dockerfile
[examples]:          https://github.com/sickp/docker-alpine-sshd/tree/master/examples/
[centos_sshd]:       https://hub.docker.com/r/sickp/centos-sshd/
