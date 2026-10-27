# Apuntes CCNA – Capa de red, enrutamiento e IPv4/IPv6 (versión corregida)

> Basados en laboratorios propios en Packet Tracer. Las secciones marcadas con ✅ estaban bien; las marcadas con 🔧 fueron corregidas.

---

## 1. Características de IP (capa 3)

- ✅ IP es **sin conexión** (cada paquete se envía de forma independiente), **de mejor esfuerzo** (*best effort*, no garantiza entrega sin errores) e **independiente del medio**.
- ✅ El método de entrega que **no garantiza** que el paquete llegue completo y sin errores es **mejor esfuerzo**. Es una característica de IP: no necesita los datos extra de control que sí usa TCP.
- 🔧 **Encapsulamiento y capas:**
  - La **capa 4 (transporte)** entrega segmentos, que se encapsulan en paquetes IP.
  - La **capa 3 (red)** agrega el encabezado IP (IP origen y destino) y es la que encapsula.
  - La **capa 2 (enlace de datos)** toma el paquete IP y lo prepara para el medio, encapsulándolo en una **trama** con MAC origen y destino.
  - Protocolos usados en esta capa: IPv4, IPv6, OSPF, ICMP.
- 🔧 **Fragmentación:** dividir un paquete IP porque el siguiente medio tiene una MTU menor se llama **fragmentación**, no segmentación. *Segmentación* es el término de TCP en la capa 4.
  - Campos de IPv4 que se usan para fragmentar: **Identification, Flags, Fragment Offset**.
  - En IPv6 los routers ya no fragmentan, y por eso esos campos no están en el encabezado base.

---

## 2. Encabezado IPv4

- ✅ Los dos campos más mencionados son **IP origen** y **IP destino** (32 bits cada uno).
- ✅ Las direcciones IP origen y destino **no cambian** mientras el paquete viaja (sin NAT).
- ✅ El campo que detecta daños es el **Header Checksum**. Cada router lo recalcula porque el TTL cambia en cada salto.
- ✅ El campo **Protocol** indica qué hay dentro: ICMP (1), TCP (6), UDP (17).

### Ejemplo real: ping a 8.8.8.8 desde Packet Tracer

| Campo | Valor |
|---|---|
| Version | 4 |
| Header Length | 20 bytes (5) |
| DSCP | 0x00 (CS0, sin prioridad especial) |
| Total Length | 60 |
| Identification | 0x19D8 |
| Flags | 0x0 |
| Fragment Offset | 0 |
| TTL | 128 |
| Protocol | ICMP (1) |
| Header Checksum | 0x0000 (validación deshabilitada) |
| Origen / Destino | 192.168.0.14 / 8.8.8.8 |

- El **checksum 0x0000** no es un error: es *checksum offloading* (la placa lo calcula después de la captura).
- TTL 128 en el request y 116 en el reply sugiere 12 saltos de vuelta. **Matiz:** el TTL inicial de Google no se conoce (puede ser 64, 128 o 255) y el camino de vuelta puede diferir del de ida, así que es una estimación.

---

## 3. IPv4 vs IPv6

- ✅ IPv6 tiene un encabezado base de **8 campos**: Version, Traffic Class, Flow Label, Payload Length, Next Header, Hop Limit, origen y destino.
- ✅ **Hop Limit** reemplaza al **TTL** y baja 1 por router.

| IPv4 | IPv6 | Observación del laboratorio |
|---|---|---|
| TTL | Hop Limit | Baja 1 por router |
| Protocol | Next Header | ICMPv6 = 0x3A (58) |
| DSCP | Traffic Class | |
| Header Checksum | No existe | IPv6 delega la verificación a capas superiores |
| Identification, Flags, Fragment Offset | No están en el encabezado base | |
| Total Length | Payload Length | PL = 108 en la captura |
| Direcciones de 32 bits | Direcciones de 128 bits | Ocupan 4 veces más |
| No existe | Flow Label | Campo nuevo |

- ✅ **Problemas de IPv4 que IPv6 resuelve:** agotamiento de direcciones, falta de conectividad de extremo a extremo (por el uso de NAT) y mayor complejidad de red.
- ✅ **Mejoras de IPv6:** mayor espacio de direcciones, mejor manejo de paquetes y eliminación de la necesidad de NAT.

---

## 4. Cómo enrutan los hosts

- ✅ Los hosts de la **misma red** se comunican sin router (usan ARP y la MAC del destino).
- ✅ Si el destino está en **otra red**, el host usa su **puerta de enlace predeterminada**: la IP del router en la red local.
- ✅ Tabla de enrutamiento en un host Windows: `netstat -r` o `route print` (sirven para IPv4 e IPv6).

### Qué cambia en cada salto (comprobado en simulación)

| Dato | ¿Cambia entre PC1 y PC3? |
|---|---|
| IP origen / destino | **No** |
| MAC origen / destino | **Sí**, en cada salto |
| TTL | **Sí**, baja 1 por router |

- PC1 → PC2 (misma red): TTL 128, ARP resuelve la MAC de PC2.
- PC1 → PC3 (otra red): **TTL 126**, porque cruza **dos routers (Rock1 y Rock2)**.
- La MAC destino que usa PC1 es la de su gateway (Rock1, `...B501`), tanto en IPv4 como en IPv6.

---

## 5. Enrutamiento en un router Cisco

- ✅ Ver la tabla: `show ip route`.
- ✅ Código **O** = ruta aprendida dinámicamente con OSPF.
- ✅ Códigos **C** (red conectada) y **L** (IP local de la interfaz):
  - `C 10.0.0.0/30`: enlace entre routers.
  - `L 10.0.0.1/32`: IP local de Rock1 en ese enlace.
  - `C 192.168.1.0/24`: LAN A.
  - `L 192.168.1.1/32`: IP de Rock1 en LAN A (gateway de PC1).
- ✅ Una ruta estática se configura **manualmente**. Un router puede combinar rutas estáticas con un protocolo dinámico (ejemplo típico: OSPF dentro de la empresa y ruta estática por defecto hacia el proveedor de internet).
- ✅ **Distancia administrativa:** si hay varias formas de llegar al mismo destino, se prefiere la de menor valor (estática = 1, OSPF = 110).

### 🔧 Ruta estática vs. ruta predeterminada

| | Ruta estática | Ruta predeterminada |
|---|---|---|
| Comando | `ip route 192.168.2.0 255.255.255.0 10.0.0.2` | `ip route 0.0.0.0 0.0.0.0 10.0.0.2` |
| Significado | "Para llegar a esta red concreta, andá por ahí" | "Todo lo que no conozca, mandalo por ahí" |
| Código | `S` | `S*` (candidata a ruta predeterminada) |
| Uso típico | Redes internas específicas | Salida hacia internet (como el router de casa con el ISP) |

- Cuando el router elige una ruta, primero usa la **coincidencia más específica (prefijo más largo)** y después compara la distancia administrativa. La ruta `0.0.0.0/0` es la menos específica, por eso se usa solo si **ninguna otra ruta coincide**. Si tampoco existe, el paquete se descarta.
- 🔧 La puerta de enlace de último recurso **no** es "la tercera opción después de fallar las primeras tres rutas".

### Lab Rock1 (resultado)

| | Antes | Después |
|---|---|---|
| Ruta | `S 192.168.2.0/24 [1/0] via 10.0.0.2` | `S* 0.0.0.0/0 [1/0] via 10.0.0.2` |
| Gateway of last resort | not set | 10.0.0.2 |

- Sin ruta hacia LAN B, el ping de PC1 a PC3 fallaba con **"Destination host unreachable"**.
- Con la ruta (estática o predeterminada) el ping funciona con **TTL 126**.
- Las líneas `C` y `L` no cambian.

---

## 6. NAT e IP públicas/privadas

- 🔧 **IP pública:** única a nivel mundial, enrutable en internet, la asigna el proveedor (ISP).
- 🔧 **IP privada** (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`): no se enruta en internet y se reutiliza en millones de redes.
- 🔧 **NAT** traduce la IP privada de origen por una pública al salir a internet.
  - Normalmente se usa **PAT** (NAT con sobrecarga): además de la IP, usa **puertos** para saber a qué equipo interno corresponde cada respuesta.
  - Con NAT sí cambia la IP origen en el borde de la red. La regla "las IP no cambian" vale para el enrutamiento sin NAT.
  - Motivo de existir: el agotamiento de direcciones IPv4.
- 🔧 La **caché DNS** y la **caché del navegador** no tienen relación con NAT. Son temas aparte: la caché DNS guarda resoluciones nombre→IP, y la del navegador guarda archivos de sitios web.
- Una IP pública no existe "para que todos la usen" ni la privada "para compartir datos": la diferencia es el alcance de enrutamiento, no la seguridad.

---

## 7. Pendientes para mejorar

- [ ] Reordenar el ejercicio PC1 → PC3: "antes de configurar" = unreachable; "después" = 4 respuestas con TTL 126.
- [ ] Verificar la redacción original de las preguntas 1 y 2 de "Características de IP" contra el libro.
- [ ] Agregar capturas de `show ip route`, `arp -a` y del encabezado en modo Simulación.
- [ ] Repetir el lab con Wireshark en GNS3 y comparar el TTL.

> ℹ️ Corregí el encabezado original "CCNP cap. 8": este contenido corresponde a **CCNA (ITN)**.
