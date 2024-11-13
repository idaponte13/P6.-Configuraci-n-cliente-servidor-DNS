# P6.-Configuraci-n-cliente-servidor-DNS

**Engade ó docker-compose do DNS outro servicio (container) que faga a función de cliente.**

O primeiro de todo foi configurar o ficheiro yaml de seguinte forma:  
```
version: '3'

services:
  asir_bind9:
    container_name: Practica6_bind9
    image: ubuntu/bind9
    platform: linux/amd64
    ports:
      - "54:53"  # Exponer el puerto 54 para que otros puedan hacer consultas DNS
    networks:
      bind9_subnet:
        ipv4_address: 172.28.5.1  # IP estática del servidor DNS
    volumes:
      - ./conf:/etc/bind/  # Montar la configuración del servidor BIND
      - ./zonas:/var/lib/bind/  # Montar el directorio de zonas
    restart: unless-stopped  # Reiniciar el contenedor si falla o si Docker se reinicia

  cliente:
    container_name: Prac6_alpine
    image: alpine
    platform: linux/amd64
    tty: true
    stdin_open: true
    dns:
      - 172.28.5.1  # El cliente usará el DNS del servidor asir_bind9
    networks:
      bind9_subnet:
        ipv4_address: 172.28.5.2  # IP estática del cliente en la misma subred
    restart: unless-stopped  # Asegurarse de que el cliente también se reinicie si falla

networks:
  bind9_subnet:
    driver: bridge  # Tipo de red 'bridge'
    ipam:
     
      config:
        - subnet: "172.28.0.0/16"  # Definir el rango de subred
          ip_range: "172.28.5.0/24"  # Rango de IPs para los contenedores
          gateway: "172.28.5.254"  # Puerta de enlace de la red          
```

Neste ficheiro configuramos un servizo DNS ```asir_bind9``` para que un cliente teña acceso pero estando dentro da mesma subrede configurado no apartado de ```networks```.  
Anterior a isto crearemos os apartados de ```conf/named.conf``` e o apartado de ```zonas/db.asircastelao.int```.

**Configura o cliente para que o seu DNS sexa o otro container, modificando el resolv.conf ou usando o fichero docker-compose.yml (preferible).**    
**Compróbao con 'dig'.**

Unha vez configurado o servidor DNS, accedemos ao cliente a través de comando `docker exec -it cliente /bin/sh`, con isto accedemos á terminal do cliente.  
Unha vez na terminal debemos facer un `apk update && apk add bind-tools`, como é alpine utilizamos apk en vez de apt, para asi poder instalar as diferentes ferreamentas de dig.

Con dig instalado faremos as consultas pertinentes para saber se está ben configurado o nosos ficheiro.  
Có comando `dig @172.28.5.1` aparecerá o seguinte codigo  

```
; <<>> DiG 9.18.27 <<>> @172.28.5.1
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: SERVFAIL, id: 40999
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 0640f1b10265dddc01000000672a7595799d1871d1fb4bae (good)
;; QUESTION SECTION:
;.				IN	NS

;; Query time: 13 msec
;; SERVER: 172.28.5.1#53(172.28.5.1) (UDP)
;; WHEN: Tue Nov 05 19:44:21 UTC 2024
;; MSG SIZE  rcvd: 56
```

Co comando `dig @172.28.5.1 test.asircastelao.int` aparecerá o seguinte:  
```
; <<>> DiG 9.18.27 <<>> @172.28.5.1 test.asircastelao.int
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 26390
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: ef382607ec522df401000000672a75c347e6194f5b451f48 (good)
;; QUESTION SECTION:
;test.asircastelao.int.		IN	A

;; ANSWER SECTION:
test.asircastelao.int.	38400	IN	A	172.28.5.4

;; Query time: 0 msec
;; SERVER: 172.28.5.1#53(172.28.5.1) (UDP)
;; WHEN: Tue Nov 05 19:45:07 UTC 2024
;; MSG SIZE  rcvd: 94  
```