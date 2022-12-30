FROM alpine:3.10
RUN mkdir /coredns
ADD ./configs/Corefile /coredns
ADD ./configs/db.dns..... /coredns
ADD ./configs/db.arpa.dns..... /coredns
ADD ./entrypoint.sh /coredns/entrypoint.sh
ADD ./coredns /bin/coredns

EXPOSE 5356/tcp 5356/udp

ENTRYPOINT ["/coredns/entrypoint.sh"]

