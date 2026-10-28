# Lab 01: Rutas estáticas y ruta predeterminada

- **Herramienta:** Cisco Packet Tracer
- **Nivel:** CCNA (enrutamiento básico)
- **Estado:** Completo (falta el análisis en Wireshark)

## 1. Objetivo
* Verificar la comunicación local (misma LAN) y remota (entre LAN A y LAN B).
* Reemplazar una ruta estática por una ruta predeterminada y comprobar que la conectividad se mantiene.
* Observar cómo cambian IP, MAC y TTL/Hop Limit en cada salto en entornos IPv4 e IPv6.

## 2. Topología y direccionamiento
Los routers se configuran con los hostnames **Rock1** y **Rock2**, conectados entre sí mediante un cable cruzado.

### Tabla de Direccionamiento IPv4

| Dispositivo | Interfaz | Dirección IP / Prefijo | Gateway Predeterminado |
| :--- | :--- | :--- | :--- |
| PC1 | Fa0 | 192.168.1.10/24 | 192.168.1.1 |
| PC2 | Fa0 | 192.168.1.20/24 | N/A |
| Rock1 | Gi0/0 | 192.168.1.1/24 | N/A (LAN A Gateway) |
| Rock1 | Gi0/1 | 10.0.0.1/30 | Enlace con Rock2 |
| Rock2 | Gi0/0 | 10.0.0.2/30 | Enlace con Rock1 |
| Rock2 | Gi0/1 | 192.168.2.1/24 | N/A (LAN B Gateway) |
| PC3 | Fa0 | 192.168.2.10/24 | 192.168.2.1 |
| Server0 | Fa0 | 192.168.2.20/24 | 192.168.2.1 |

### Tabla de Direccionamiento IPv6

| Dispositivo | Interfaz | Dirección IPv6 / Prefijo | Gateway Predeterminado |
| :--- | :--- | :--- | :--- |
| PC1 | Fa0 | 2001:DB8:A::10/64 | 2001:DB8:A::1 |
| Rock1 | Gi0/0 | 2001:DB8:A::1/64 | N/A |
| Rock1 | Gi0/1 | 2001:DB8:FF::1/64 | Enlace con Rock2 |
| Rock2 | Gi0/0 | 2001:DB8:FF::2/64 | Enlace con Rock1 |
| Rock2 | Gi0/1 | 2001:DB8:B::1/64 | N/A |
| PC3 | Fa0 | 2001:DB8:B::10/64 | 2001:DB8:B::1 |

## 3. Comunicación Local y Remota
* **Comunicación Local (PC1 → PC2):** El paquete no cruza ningún router. El valor de **TTL es 128** e IP/MAC origen y destino se mantienen dentro de la misma LAN A.
* **Comunicación Remota (PC1 → PC3):** Al atravesar los routers Rock1 y Rock2, el paquete experimenta dos saltos, por lo que llega a destino con un **TTL de 126** (128 - 2).

## 4. Configuración y Cambio de Ruta en Rock1
Se retiró la ruta estática específica hacia la LAN B y se implementó una ruta estática predeterminada (`0.0.0.0/0`) apuntando al siguiente salto. También se habilitó el procesamiento global de IPv6:

```cisconet
! Configuración en Rock1:
interface GigabitEthernet0/0
 ip address 192.168.1.1 255.255.255.0
 ipv6 address 2001:DB8:A::1/64
!
interface GigabitEthernet0/1
 ip address 10.0.0.1 255.255.255.252
 ipv6 address 2001:DB8:FF::1/64
!
ipv6 unicast-routing
ip route 0.0.0.0 0.0.0.0 10.0.0.2
ipv6 route 2001:DB8:B::/64 2001:DB8:FF::2
```

## 5. Qué cambia en cada salto (Conclusiones)
* **Direcciones IP:** Las direcciones IP origen y destino **no cambian** a lo largo del viaje del paquete.
* **Direcciones MAC:** Las direcciones MAC **cambian en cada salto** de Capa 2 para poder entregar la trama al siguiente nodo conectado directamente.
* **TTL / Hop Limit:** El valor decrementa en 1 unidad por cada router (dispositivo de Capa 3) que procesa y reenvía el paquete.
* **Ruta Predeterminada:** Simplifica la tabla de enrutamiento en routers de borde, derivando todo el tráfico desconocido hacia el siguiente salto.

## 6. Relación con Seguridad
* Una ruta predeterminada mal configurada puede generar bucles de enrutamiento o desviar tráfico interno sensible hacia interfaces o redes no deseadas.
* Auditar las tablas de rutas (`show ip route`) constituye una de las primeras acciones defensivas ante incidentes de redirección de tráfico anómalo.

## 7. Pendientes
- [ ] Agregar el archivo del laboratorio `.pkt` sanitizado.
- [ ] Incorporar los archivos planos de configuración limpia en la carpeta `configs/`.
- [ ] Replicar la topología en GNS3 para capturar el tráfico real con Wireshark usando filtros `icmp`, `arp` e `icmpv6`.


