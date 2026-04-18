# /cisco-pt — Cisco Packet Tracer MCP Skill

> Skill autosuficiente para crear, configurar y validar topologías de red en Cisco Packet Tracer usando el MCP `packet-tracer`. Cubre modo Normal (laboratorio libre) y modo Examen (bajo presión académica).

## Infraestructura

- **MCP**: `packet-tracer` — ÚNICA herramienta necesaria
- **Ubicación bridge**: `C:\Users\Andrea\Desktop\MCP-Packet-Tracer\`
- **Versión**: v0.4.0 · 25 tools · `python -m packet_tracer_mcp --stdio`
- **Repo**: `MauricioFonck/Mcp-Cisco-Paker-Tracer`
- **Defaults obligatorios**: Router = `Router-PT` · Switch = `2960-24TT`
- **Ejecución**: DIRECTA — sin agentes, solo MCP `packet-tracer`

---

## Detección de Modo

| Señales MODO NORMAL | Señales MODO EXAMEN |
|---------------------|---------------------|
| "topología", "red", "router", "switch", "cisco", "VLAN", "IP", "routing", "packet tracer", "simula red", "crea una topología", "construye red" | "modo examen", "estoy en examen", "parcial en clase", "analiza lo que armé", "ya armé la topología", "listo analiza", "configura lo que ya está armado" |

---

## MODO NORMAL

### A) Topología nueva desde cero

```
→ Recopilar requisitos (routers, PCs, routing, DHCP, NAT...)
→ pt_full_build(routers=N, pcs_per_lan=N, routing="...", router_model="Router-PT", deploy=true)
→ Si necesita NAT/ACL (no soportados por generador):
  → pt_send_raw('configureIosDevice("RouterX", "...comandos NAT...")')
```

### B) Configuración de topología existente

```
→ pt_query_topology() → escanear dispositivos
→ Mostrar lo encontrado, preguntar qué configurar
→ Armar bloque CLI por router (NAT, helper-address, etc.)
→ pt_send_raw('configureIosDevice("RouterX", "enable\nconfigure terminal\n...")')
→ Para PC-PT/Laptop-PT: pt_send_raw('configurePcIp("PC0", true)')
→ Para Server-PT: pt_configure_server_static(device="Server0", ip="...", mask="...", gateway="...")
→ Explicar lo hecho, sugerir verificaciones
```

---

## MODO EXAMEN — 4 Fases (Velocidad máxima)

> Contexto: solo PT + CMD abiertas, profesor vigila, tiempo límite. El usuario armó la topología manualmente. Claude configura lo que el usuario dicta.

**Reglas absolutas:**
- ⛔ NUNCA `pt_full_build` ni `pt_live_deploy` — crearían duplicados
- Cero texto innecesario — solo acción y confirmación breve
- Preguntar ANTES de ejecutar si hay conflicto crítico
- NUNCA mezclar configuración de routers y activación DHCP en el mismo paso

---

### FASE 1 — ESCANEO

```
pt_query_topology() → nombres y modelos reales
```

Escaneo de enlaces (NO usar pt_query_links — da timeout):
```javascript
var n=ipc.network();var dc=n.getDeviceCount();var linksMap={};
for(var i=0;i<dc;i++){var d=n.getDeviceAt(i);if(!d)continue;
var dn=d.getName();var pc=d.getPortCount();
for(var j=0;j<pc;j++){var p=d.getPortAt(j);
if(p&&p.getLink()){var uuid=p.getLink().getObjectUuid();
var entry=dn+'['+p.getName()+']';
if(!linksMap[uuid]){linksMap[uuid]=entry;}
else{linksMap[uuid]=linksMap[uuid]+'<--->'+entry;}}}}
var results=[];for(var k in linksMap){results.push(linksMap[k]);}
reportResult(results.length>0?results.join('|'):'sin_enlaces');
```
Parsear con separador `|`. Mostrar: `"Detecté: [dispositivos] con [N] enlaces."`

---

### FASE 2 — INTERPRETAR Y VALIDAR

El usuario dicta config del tablero/hoja. Puede ser caótica o con errores tipográficos.

**PASO A — EXTRAER (siempre primero):**

Patrones reconocidos durante extracción:
- `"DATO IMPORTANTE:"` / `"CORRECCIÓN:"` → prioridad máxima
- Tabla VLSM (subred base + host counts) → calcular ANTES de asignar IPs (mayor a menor)
- DHCP multi-fuente (`"PC0→DHCP del Router2"`) → mapear `{dispositivo → fuente}`
- VLANs con rangos de puertos → mapear puertos y fuente DHCP por VLAN
- NAT global (`"RouterX debe brindar NAT"`) → inside/outside + ruta default en todos
- IP estática mezclada con DHCP → distinguir explícitamente
- Inconsistencia header/cuerpo → flagear `[CONFLICTO]`

**PASO B — COMPLETAR (solo lo verdaderamente ausente):**

| Dato ausente | Default |
|---|---|
| IPs no dadas | Subredes VLSM calculadas; si no: `192.168.X.0/24` LANs, `10.0.X.X/30` WAN |
| DNS | `8.8.8.8` |
| Clock rate | `64000` (seriales) |
| DHCP no mencionado | Asumir SÍ |
| Routing no especificado | **PREGUNTAR** — no asumir |

⚠️ Si el dato existe aunque mal escrito → NO usar default, corregirlo con `[corregido]`

**PASO C — IR DIRECTO A EJECUCIÓN (sin confirmación en MODO EXAMEN):**

Mostrar resumen de 3 líneas antes de inyectar:
```
"Configurando: Router0 Fa0/0=10.x.x.1/24, Se2/0=10.0.0.1/30 | RIP V2 | NAT en Router0."
```
Solo pausar si hay `[CONFLICTO CRÍTICO]`:
```
"⚠️ [pregunta de 1 línea — ej: ¿NAT en Router0 o Router1?]"
```
Routing ausente → única excepción donde preguntar antes de asumir.

---

### FASE 3 — PASO A: CONFIGURAR ROUTERS + IPs ESTÁTICAS (paralelo)

⛔ NO activar DHCP en los hosts aquí.

**En UN SOLO mensaje paralelo — GRUPO A + GRUPO B simultáneos:**

**GRUPO A — Routers** (todos en paralelo):
```javascript
configureIosDevice("RouterX", "enable\nconfigure terminal\n[bloque CLI completo]")
```

**GRUPO B — Dispositivos con IP estática** (en paralelo con routers):
```
⚠️ BUG CONOCIDO: configurePcIp() NO funciona en Server-PT

Para Server-PT (solo IP):
  pt_configure_ip(device="Server0", ip="x.x.x.x", mask="...", gateway="...")

Para Server-PT (IP + HTTP o DHCP pool):
  pt_configure_server_static(device="Server0", ip="x.x.x.x", mask="...", gateway="...", http_enabled=True)

Para PC-PT/Laptop-PT con IP estática:
  pt_send_raw('configurePcIp("PC0", false, "x.x.x.x", "255.x.x.x", "x.x.x.x")')
```

**Reglas de blindaje anti-fallos obligatorias:**
- `clock rate 64000` → SIEMPRE en interfaces seriales (agregar en ambos lados — PT aplica solo en DCE)
- STP wait → si hay Switch: incluir `time.sleep(30)` en script Python después de activar puertos
- Nombre lógico CLI: usar el nombre real que devolvió el escaneo de enlaces
  (ej: si el API dijo `"FastEthernet0/0"`, usar `"Fa0/0"` en el bloque CLI)

**⚠️ MENSAJE OBLIGATORIO AL USUARIO después de este paso:**
```
"¡IMPORTANTE! Abre manualmente la pestaña 'CLI' de cada router en PT al menos una vez
 para despertar el motor IOS. Cuando termines, respóndeme LISTO."
```

---

### FASE 4 — PASO B: ACTIVAR DHCP EN HOSTS (solo tras confirmación)

→ Solo ejecutar cuando el usuario responda **"LISTO"**
→ Solo para hosts con DHCP (PCs, Laptops sin IP estática)

Renovar IP 2 veces para forzar nueva solicitud:

```javascript
// Limpiar primero (todos en paralelo)
configurePcIp("PC0", false, "0.0.0.0", "0.0.0.0", "0.0.0.0")
configurePcIp("Laptop0", false, "0.0.0.0", "0.0.0.0", "0.0.0.0")

// Luego activar DHCP (todos en paralelo)
configurePcIp("PC0", true)
configurePcIp("Laptop0", true)
```

Para Server-PT con DHCP (`configurePcIp` NO funciona):
```
pt_configure_ip(device="Server0", dhcp=True)
```

Mensaje final: `"¡Red operativa! Verifica con ping [gateway] desde cada PC."`

---

## Funciones JS disponibles en pt_send_raw

| Función | Para qué | Limitaciones |
|---|---|---|
| `configureIosDevice(name, cliBlock)` | CLI IOS completo a router/switch | Solo routers y switches |
| `configurePcIp(name, dhcp)` | DHCP en PC-PT y Laptop-PT | ❌ NO funciona en Server-PT |
| `configurePcIp(name, false, ip, mask, gw)` | IP estática en PC-PT y Laptop-PT | ❌ NO funciona en Server-PT |
| `addDevice(name, model, x, y)` | Agregar dispositivo | Solo topología nueva |
| `addLink(dev1, port1, dev2, port2)` | Agregar enlace | Solo topología nueva |
| `reportResult(value)` | Devolver resultado (wait_result=True) | Obligatorio para leer output |

## Tools MCP dedicadas para end-devices

| Tool | Para qué | Cuándo usar |
|---|---|---|
| `pt_configure_ip` | IP en cualquier dispositivo (PC-PT, Server-PT, Laptop-PT) | Server-PT cuando solo se necesita la IP |
| `pt_configure_server_static` | Server-PT con IP + servicios opcionales | Server-PT con HTTP o pool DHCP |

---

## Referencia VLSM rápida

Calcular de mayor a menor número de hosts:

| Hosts requeridos | Bits host | Prefijo | Hosts útiles | Máscara |
|---|---|---|---|---|
| 1-2 | 2 | /30 | 2 | 255.255.255.252 |
| 3-6 | 3 | /29 | 6 | 255.255.255.248 |
| 7-14 | 4 | /28 | 14 | 255.255.255.240 |
| 15-30 | 5 | /27 | 30 | 255.255.255.224 |
| 31-62 | 6 | /26 | 62 | 255.255.255.192 |
| 63-126 | 7 | /25 | 126 | 255.255.255.128 |
| 127-254 | 8 | /24 | 254 | 255.255.255.0 |
| 255-510 | 9 | /23 | 510 | 255.255.254.0 |
| 511-1022 | 10 | /22 | 1022 | 255.255.252.0 |
| 1023-2046 | 11 | /21 | 2046 | 255.255.248.0 |
| 2047-4094 | 12 | /20 | 4094 | 255.255.240.0 |

---

## Plantillas CLI por protocolo

### RIP V2
```
router rip
 version 2
 network [red-directamente-conectada]
 no auto-summary
 default-information originate   ← solo en router con ruta default
```

### NAT Overload (PAT)
```
interface [inside-interface]
 ip nat inside
interface [outside-interface]
 ip nat outside
access-list 1 permit [red-interna] [wildcard]
ip nat inside source list 1 interface [outside-interface] overload
ip route 0.0.0.0 0.0.0.0 [next-hop]
```

### DHCP en Router
```
ip dhcp excluded-address [gateway]
ip dhcp pool [NOMBRE]
 network [red] [mascara]
 default-router [gateway]
 dns-server 8.8.8.8
```

### DHCP Relay (helper-address)
```
interface [LAN-interface]
 ip helper-address [IP-servidor-DHCP]
```

### VLANs en Switch
```
vlan [ID]
 name [NOMBRE]
interface [puerto-acceso]
 switchport mode access
 switchport access vlan [ID]
```

### OSPF
```
router ospf 1
 network [red] [wildcard] area [N]
 passive-interface [LAN-interface]
```

### EIGRP
```
router eigrp [AS]
 network [red] [wildcard]
 no auto-summary
```

---

## Reglas clave (nunca omitir)

1. Siempre `Router-PT` y `Switch 2960-24TT`
2. `clock rate 64000` en TODOS los seriales (PT aplica solo en DCE, ignorar en DTE no rompe nada)
3. STP 30s — esperar antes de pedir DHCP cuando hay Switches
4. Si faltan puertos → pausar, indicar qué módulo agregar manualmente en PT
5. NAT/ACL/VLANs/helper-address → incluir en bloque CLI de `configureIosDevice`
6. NUNCA spawning de agentes — solo MCP directo
7. NUNCA inventar datos — preguntar si falta algo crítico
8. NUNCA usar tools que no existen en el MCP

---

## Protocolo ante fallo de bridge

Si `pt_bridge_status` falla o PT no responde:
```
1. Verificar que Packet Tracer esté abierto y con una topología activa
2. Verificar bridge en: C:\Users\Andrea\Desktop\MCP-Packet-Tracer\
3. Si sigue sin conectar → usuario ejecuta: python -m packet_tracer_mcp --stdio
4. Reintentar pt_bridge_status
```
