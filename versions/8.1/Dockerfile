FROM alpine:3.10
LABEL maintainer "Adrian B. Danieli - https://github.com/sickp - Owen Warner added 8.1"
ENTRYPOINT ["/entrypoint.sh"]
EXPOSE 22
COPY rootfs /

RUN apk add --no-cache openssh \
  && sed -i s/#PermitRootLogin.*/PermitRootLogin\ yes/ /etc/ssh/sshd_config \
  && echo "root:root" | chpasswd
