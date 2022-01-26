FROM ubuntu:focal-20220105

RUN apt-get update
RUN apt-get install -y haproxy && apt-get clean

STOPSIGNAL SIGUSR1

COPY docker-entrypoint.sh /usr/local/bin/
COPY haproxy.cfg /etc/haproxy/haproxy.cfg
RUN ln -s /usr/local/bin/docker-entrypoint.sh / # backwards compat
RUN ["chmod", "+x", "/usr/local/bin/docker-entrypoint.sh"]
ENTRYPOINT ["docker-entrypoint.sh"]
#
# no USER for backwards compatibility (to try to avoid breaking existing users)
CMD ["haproxy", "-f", "/etc/haproxy/haproxy.cfg"]
