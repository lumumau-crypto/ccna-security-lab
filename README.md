# Lab 01: Rutas estáticas y ruta predeterminada

**Herramienta:** Cisco Packet Tracer

**Nivel:** CCNA (enrutamiento básico)

**Estado:** Completo (falta el análisis en Wireshark)

## 1. Objetivo

- Verificar la comunicación local (misma LAN) y remota (entre LAN A y LAN B).
- Reemplazar una ruta estática por una ruta predeterminada y comprobar que la conectividad se mantiene.
- Observar cómo cambian IP, MAC y TTL en cada salto.

## 2. Topología y direccionamiento

Topología *Los routers se ven como Router1 y Router2 en el diagrama (nombre de pantalla); sus hostnames configurados son Rock1 y Rock2. La línea punteada entre ellos es un cable cruzado.*

| Dispositivo | Interfaz | Dirección |
| --- | --- | --- |
| PC1 | Fa0 | 192.168.1.10/24, GW 192.168.1.1 |
| PC2 | Fa0 | 192.168.1.20/24 |
| Rock1 | Gi0/0 | 192.168.1.1/24 (LAN A) |
| Rock1 | Gi0/1 | 10.0.0.1/30 (enlace con Rock2) |
| Rock2 | Gi0/0 | 10.0.0.2/30 (enlace con Rock1) |
| Rock2 | Gi0/1 | 192.168.2.1/24 (LAN B) |
| PC3 | Fa0 | 192.168.2.10/24 (LAN B) |
| Server0 | Fa0 | 192.168.2.20/24, GW 192.168.2.1 |
| Switch0 | 2960-24TT | Conecta PC1, PC2 y Rock1 (LAN A) |
| Switch1 | 2960-24TT | Conecta Rock2, PC3 y Server0 (LAN B) |

Configuración de interfaces de Rock2:

Rock2 show ip interface brief

## 3. Comunicación local: PC1 → PC2

PC1 envía el ping a 192.168.1.20 y aprende la MAC de PC2 por ARP.

PC1 ping PC2

- TTL = 128: el paquete no cruza ningún router.
- La tabla ARP de PC1 pasa a tener dos entradas: el gateway (192.168.1.1) y PC2 (192.168.1.20).

ipconfig y ARP

## 4. Comunicación remota: PC1 → PC3

PC1 ping PC3

- **Sin ruta hacia LAN B** en Rock1: el ping falla con *Destination host unreachable*.
- **Con ruta hacia LAN B:** 4 respuestas con **TTL = 126**.
- 128 − 126 = 2 saltos: el paquete atraviesa **Rock1 y Rock2**.

## 5. Cambio realizado en Rock1

Se reemplazó la ruta estática por una ruta predeterminada:

```
Rock1> enable
Rock1# configure terminal
Rock1(config)# no ip route 192.168.2.0 255.255.255.0 10.0.0.2
Rock1(config)# ip route 0.0.0.0 0.0.0.0 10.0.0.2
Rock1(config)# end
Rock1# show ip route
```

show ip route

|  | Antes | Después |
| --- | --- | --- |
| Ruta | `S 192.168.2.0/24 [1/0] via 10.0.0.2` | `S* 0.0.0.0/0 [1/0] via 10.0.0.2` |
| Gateway of last resort | not set | 10.0.0.2 |

Las rutas conectadas (`C`) y locales (`L`) no cambian.

## 6. Qué cambia en cada salto

| Dato | ¿Cambia? |
| --- | --- |
| IP origen/destino | No |
| MAC origen/destino | Sí, en cada salto |
| TTL | Sí, baja 1 por router |

Observado en modo Simulación: la MAC destino que usa PC1 es la de Rock1 (gateway), y en el tramo siguiente cambian ambas MAC.

## 7. Conclusiones

- Una ruta estática indica cómo llegar a una red concreta. La predeterminada (`0.0.0.0/0`) indica qué hacer con todo lo que el router no conoce.
- El router prefiere siempre la coincidencia más específica; la ruta predeterminada solo se usa si no hay otra.
- La ruta predeterminada simplifica la configuración en routers de borde, pero es menos precisa y puede ocultar errores de enrutamiento.

## 8. Pendientes

- \[ \] Agregar el archivo `.pkt`
- \[ \] Agregar `configs/Rock1.txt` y `configs/Rock2.txt` (sin contraseñas reales)
- \[ \] Repetir el lab en GNS3 y capturar con Wireshark (filtros `icmp` y `arp`)

## 9. Relación con seguridad

- Una ruta predeterminada mal configurada puede enviar tráfico interno a un destino no deseado.
- Revisar la tabla de enrutamiento es el primer paso al investigar desvíos de tráfico.
