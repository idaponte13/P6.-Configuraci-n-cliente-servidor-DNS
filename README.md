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
