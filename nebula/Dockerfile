FROM ubuntu
RUN mkdir /opt/nebula
ADD ./nebula /bin/nebula
EXPOSE 4242/udp
ENTRYPOINT ["/bin/nebula"]