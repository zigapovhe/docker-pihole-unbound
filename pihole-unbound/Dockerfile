FROM pihole/pihole:2022.09.2
RUN apt update && apt install -y unbound

COPY lighttpd-external.conf /etc/lighttpd/external.conf 
RUN echo "include: \"/opt/unbound/*.conf\"" >> /etc/unbound/unbound.conf 
COPY 99-edns.conf /etc/dnsmasq.d/99-edns.conf
COPY start_unbound_and_s6_init.sh start_unbound_and_s6_init.sh

RUN chmod +x start_unbound_and_s6_init.sh
ENTRYPOINT ./start_unbound_and_s6_init.sh
