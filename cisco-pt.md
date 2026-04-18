# /cisco-pt — Cisco Packet Tracer MCP Skill

> Skill autosuficiente para crear, configurar y validar topologías de red en Cisco Packet Tracer usando el MCP `packet-tracer`. Cubre modo Normal (laboratorio libre) y modo Examen (bajo presión académica).

## Infraestructura

- **MCP**: `packet-tracer` — ÚNICA herramienta necesaria
- **Ubicación bridge**: `C:\Users\Andrea\Desktop\MCP-Packet-Tracer\`
- **Versión**: v0.4.0 · 26 tools · 5 resources
- **Transporte**: `http://127.0.0.1:39000/mcp` (default) | `--stdio` para debug
- **Bridge interno**: `http://127.0.0.1:54321` (PT ↔ MCP)
- **Repo**: `MauricioFonck/Mcp-Cisco-Paker-Tracer`
- **Ejecución**: DIRECTA — sin agentes, solo MCP `packet-tracer`

### Preferencias del usuario (siempre aplicar)

- **Router**: `Router-PT` — el usuario los coloca y configura manualmente en PT
- **Switch**: `2960-24TT`
- **Cables router↔router**: seriales en la mayoría de los casos → siempre incluir `clock rate 64000`
- **Flujo habitual**: usuario arma la topología a mano → pide escaneo → yo configuro
- **Puertos**: NUNCA asumir qué puertos tiene un router — siempre leer del escaneo
  (el usuario agrega módulos HWIC-2T manualmente, puede tener cualquier combinación de puertos seriales/GigE)
- **Responsabilidad del usuario**: agregar hardware, colocar dispositivos, conectar cables
- **Responsabilidad de Claude**: escanear topología, reportar con puertos exactos, inyectar CLI

### Arranque requerido (verificar ANTES de cualquier operación)

```
1. python -m src.packet_tracer_mcp  (desde C:\Users\Andrea\Desktop\MCP-Packet-Tracer\)
2. Packet Tracer abierto con topología activa
3. Extensions > Builder Code Editor → pegar bootstrap → Run (UNA vez por sesión)
4. pt_bridge_status → debe responder conectado
```

Bootstrap (pegar en Builder Code Editor):
```javascript
/* PT-MCP Bridge */ window.webview.evaluateJavaScriptAsync("setInterval(function(){var x=new XMLHttpRequest();x.open('GET','http://127.0.0.1:54321/next',true);x.onload=function(){if(x.status===200&&x.responseText){$se('runCode',x.responseText)}};x.onerror=function(){};x.send()},500)");
```

---

## Detección de Modo

| Señales MODO NORMAL | Señales MODO EXAMEN |
|---------------------|---------------------|
| "topología", "red", "router", "switch", "cisco", "VLAN", "IP", "routing", "packet tracer", "simula red", "crea una topología", "construye red" | "modo examen", "estoy en examen", "parcial en clase", "analiza lo que armé", "ya armé la topología", "listo analiza", "configura lo que ya está armado" |

---

## Catálogo de dispositivos soportados

### Routers (pt_type)

| Modelo | pt_type | Puertos físicos | Serie por default |
|--------|---------|-----------------|-------------------|
| 1941 | `1941` | GigabitEthernet0/0, GigabitEthernet0/1 | ❌ requiere HWIC-2T |
| 2901 | `2901` | GigabitEthernet0/0, GigabitEthernet0/1 | ❌ requiere HWIC-2T |
| 2911 | `2911` | GigabitEthernet0/0, GigabitEthernet0/1, GigabitEthernet0/2 | ❌ requiere HWIC-2T |
| ISR 4321 | `ISR4321` | **GigabitEthernet0/0/0**, **GigabitEthernet0/0/1** | ❌ requiere HWIC-2T |

> ⚠️ CRÍTICO: `Router-PT` NO está en el catálogo del MCP. Usar siempre `2911` para el generador automático.
> ⚠️ CRÍTICO: ISR4321 usa notación `0/0/0` no `0/0` — error frecuente al escribir CLI.
> ⚠️ CRÍTICO: Ningún router tiene puertos seriales por defecto. Para WAN serial: agregar módulo HWIC-2T manualmente en PT (slot 0 para 2911, puede fallar en otros).

### Switches (pt_type)

| Modelo | pt_type | Puertos FastEthernet | Puertos GigabitEthernet |
|--------|---------|----------------------|------------------------|
| 2960 | `2960-24TT` | Fa0/1–Fa0/24 | Gi0/1, Gi0/2 |
| 3560 (L3) | `3560-24PS` | Fa0/1–Fa0/24 | Gi0/1, Gi0/2 |

### End Devices

| Modelo | pt_type | Puerto principal | Configuración |
|--------|---------|-----------------|---------------|
| PC | `PC-PT` | FastEthernet0 | `configurePcIp()` ✅ |
| Laptop | `Laptop-PT` | FastEthernet0 | `configurePcIp()` ✅ |
| Server | `Server-PT` | FastEthernet0 | `configurePcIp()` ❌ — usar `pt_configure_server_static` |

### Otros

| Modelo | pt_type | Notas |
|--------|---------|-------|
| Cloud | `Cloud-PT` | Solo `Ethernet6` para Ethernet normal. Serial0-3/Modem4-5/Coaxial7 = uso especial |
| Access Point | `AccessPoint-PT` | Port 0, Port 1 |

---

## Reglas de cable (cable_type)

| Conexión | Cable |
|----------|-------|
| Router ↔ Switch | `straight` |
| Switch ↔ PC / Server / Laptop | `straight` |
| Switch ↔ Access Point | `straight` |
| Router ↔ Router (directo) | `cross` |
| Switch ↔ Switch | `cross` |
| Router ↔ PC (directo, sin switch) | `cross` |
| Router ↔ Cloud | `straight` |
| Router ↔ Router (serial/WAN) | `serial` (requiere HWIC-2T en ambos routers) |

---

## Limitaciones críticas del MCP (no inventar lo que no existe)

| Feature | Estado |
|---------|--------|
| Routing static | ✅ Implementado |
| Routing OSPF | ✅ Implementado |
| Routing RIP | ❌ Solo enum — NO genera config. Usar `configureIosDevice` con CLI manual |
| Routing EIGRP | ❌ Solo enum — NO genera config. Usar `configureIosDevice` con CLI manual |
| NAT / PAT | ❌ No generado automáticamente — incluir en bloque CLI de `configureIosDevice` |
| ACLs | ❌ No generadas automáticamente — incluir en bloque CLI de `configureIosDevice` |
| VLANs | ❌ No generadas automáticamente — incluir en bloque CLI de `configureIosDevice` |
| Puertos seriales | ❌ No hay por defecto — requiere módulo HWIC-2T agregado manualmente |
| HWIC-2T por API | ⚠️ `addModule` puede fallar según slot y modelo (probado: falla en 2911 slot 1) |

---

## Referencia completa de 26 tools MCP

### Consulta de catálogo
| Tool | Parámetros | Uso |
|------|-----------|-----|
| `pt_list_devices` | — | Lista todos los dispositivos con puertos. Usar antes de armar topología. |
| `pt_list_templates` | — | Lista plantillas predefinidas (single_lan, multi_lan, star, etc.) |
| `pt_get_device_details` | `model` | Detalles completos de un modelo específico |

### Planificación
| Tool | Parámetros clave | Uso |
|------|-----------------|-----|
| `pt_estimate_plan` | igual que plan | Dry-run rápido: estima sin generar plan |
| `pt_plan_topology` | `routers`, `pcs_per_lan`, `routing`, `router_model`, `switch_model`, `has_wan` | Genera plan completo |

### Validación y corrección
| Tool | Parámetros | Uso |
|------|-----------|-----|
| `pt_validate_plan` | `plan` | Valida con 15 tipos de error tipificados |
| `pt_fix_plan` | `plan` | Auto-corrige cables, modelos, puertos |
| `pt_explain_plan` | `plan` | Explica decisiones en lenguaje natural |

### Generación
| Tool | Parámetros | Uso |
|------|-----------|-----|
| `pt_generate_script` | `plan` | Genera script JS para PTBuilder |
| `pt_generate_configs` | `plan` | Genera bloques CLI por dispositivo |

### Pipeline completo
| Tool | Parámetros clave | Uso |
|------|-----------------|-----|
| `pt_full_build` | `routers`, `pcs_per_lan`, `routing`, `router_model="2911"`, `switch_model="2960-24TT"`, `deploy=true/false` | Todo en uno: planifica → valida → genera → despliega |

### Despliegue
| Tool | Parámetros | Uso |
|------|-----------|-----|
| `pt_bridge_status` | — | Verifica si el bridge está activo ← SIEMPRE primero |
| `pt_deploy` | `plan` | Copia script al portapapeles (despliegue manual) |
| `pt_live_deploy` | `plan` | Envía a PT en tiempo real via bridge |

### Interacción con topología existente
| Tool | Parámetros | Uso |
|------|-----------|-----|
| `pt_query_topology` | — | Consulta dispositivos en PT. Retorna nombre, tipo, modelo y posición de cada dispositivo. |
| `pt_query_links` | — | Consulta todos los enlaces activos en PT. Retorna el mapeo de conexiones `DevA[portA] <---> DevB[portB]`. Preferir sobre el bucle JS manual. |
| `pt_delete_device` | `device_name` | Elimina dispositivo y sus enlaces automáticamente |
| `pt_rename_device` | `old_name`, `new_name` | Renombra dispositivo en PT |
| `pt_move_device` | `device_name`, `x`, `y` | Mueve a coordenadas (x, y) en el canvas |
| `pt_delete_link` | `device_name`, `interface_name` | Elimina el enlace conectado a una interfaz específica |
| `pt_send_raw` | `js_code`, `wait_result` | Envía JS arbitrario al Script Engine de PT. `wait_result=True` espera respuesta via `reportResult()` |

### Configuración de end-devices
| Tool | Parámetros | Uso |
|------|-----------|-----|
| `pt_configure_ip` | `device`, `dhcp`, `ip`, `mask`, `gateway`, `dns` | Configura IP (estática o DHCP) en cualquier end-device: PC-PT, Laptop-PT, Server-PT. Usa API IPC directa del puerto — funciona en todos los tipos incluyendo Server-PT donde `configurePcIp()` falla silenciosamente. |
| `pt_configure_server_static` | `device`, `ip`, `mask`, `gateway`, `dns`, `http_enabled`, `dhcp_pool_start`, `dhcp_pool_end` | Especializada para Server-PT: configura IP estática y opcionalmente activa servicio HTTP y/o configura pool DHCP gestionado por el servidor. |

### Export y proyectos
| Tool | Parámetros | Uso |
|------|-----------|-----|
| `pt_export` | `plan`, `output_dir` | Exporta plan + scripts + configs a archivos |
| `pt_list_projects` | — | Lista proyectos guardados |
| `pt_load_project` | `project_name` | Carga proyecto guardado |

---

## Funciones JS disponibles en pt_send_raw

| Función | Firma completa | Limitaciones |
|---------|---------------|--------------|
| `configureIosDevice(name, cliBlock)` | `configureIosDevice("R1", "enable\nconfigure terminal\n...")` | Solo routers y switches |
| `configurePcIp(name, dhcp)` | `configurePcIp("PC0", true)` | DHCP en PC-PT y Laptop-PT. ❌ NO Server-PT |
| `configurePcIp(name, false, ip, mask, gw, dns)` | `configurePcIp("PC0", false, "192.168.1.2", "255.255.255.0", "192.168.1.1", "8.8.8.8")` | IP estática. ❌ NO Server-PT |
| `addDevice(name, model, x, y)` | `addDevice("R1", "2911", 100, 200)` | Solo topología nueva |
| `addLink(dev1, port1, dev2, port2)` | `addLink("R1", "GigabitEthernet0/0", "SW1", "GigabitEthernet0/1", "straight")` | Solo topología nueva |
| `queryTopology()` | `queryTopology()` | Retorna via `reportResult`. Requiere `wait_result=True` |
| `deleteDevice(name)` | `deleteDevice("R1")` | Requiere `wait_result=True` |
| `renameDevice(old, new)` | `renameDevice("Router0", "R1")` | Requiere `wait_result=True` |
| `moveDevice(name, x, y)` | `moveDevice("R1", 300, 200)` | Requiere `wait_result=True` |
| `deleteLink(device, iface)` | `deleteLink("R1", "GigabitEthernet0/0")` | Requiere `wait_result=True` |
| `reportResult(value)` | `reportResult("ok")` | Obligatorio para leer output |

> ⚠️ La API del Script Engine NO tiene XMLHttpRequest. Solo el webview lo tiene.
> ⚠️ `for(var p in port)` devuelve vacío — los métodos de puerto son C++ no enumerables.
> ⚠️ Nombres de puertos exactos — usar el nombre real devuelto por escaneo, no abreviaciones.

## MODO NORMAL

### A) Topología nueva desde cero

```
→ pt_bridge_status() → verificar conexión
→ Recopilar requisitos (routers, PCs, routing, DHCP, NAT...)
→ ADVERTENCIA: RIP y EIGRP NO son generados automáticamente — si los pide, usar CLI manual
→ pt_full_build(routers=N, pcs_per_lan=N, routing="static"|"ospf", router_model="2911",
                switch_model="2960-24TT", deploy=true)
→ Si necesita NAT/ACL/VLAN/RIP/EIGRP (no soportados por generador):
  → pt_send_raw('configureIosDevice("RouterX", "...comandos CLI...")')
```

### B) Configuración de topología existente

```
→ pt_bridge_status() → verificar conexión
→ pt_query_topology() → escanear dispositivos y nombres reales
→ pt_query_links() → escanear enlaces activos (preferir sobre bucle JS manual)
→ Mostrar lo encontrado, preguntar qué configurar
→ Armar bloque CLI por router (NAT, helper-address, VLANs, etc.)
→ pt_send_raw('configureIosDevice("RouterX", "enable\nconfigure terminal\n...")')
→ Para PC-PT/Laptop-PT con DHCP: pt_send_raw('configurePcIp("PC0", true)')
→ Para PC-PT/Laptop-PT con IP estática: pt_send_raw('configurePcIp("PC0", false, "ip", "mask", "gw", "dns")')
→ Para Server-PT (solo IP): pt_configure_ip(device="Server0", ip="...", mask="...", gateway="...")
→ Para Server-PT (IP + HTTP o DHCP): pt_configure_server_static(device="Server0", ip="...", ...)
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

### FASE 1 — ESCANEO (siempre es el primer paso)

> El usuario siempre pide escanear antes de configurar. Reportar TODO con puertos exactos.
> NUNCA asumir qué puertos tiene un Router-PT — el usuario los configura manualmente con módulos.

```
pt_query_topology() → nombres y modelos reales
```

Escaneo de enlaces alternativo con JS manual (solo si pt_query_links falla o da timeout):
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
Parsear con separador `|`.

**Formato de reporte al usuario (obligatorio):**
```
Dispositivos detectados:
  R1 (Router-PT) — puertos: Serial0/1/0, Serial0/1/1, GigabitEthernet0/0
  R2 (Router-PT) — puertos: Serial0/1/0, GigabitEthernet0/0
  SW1 (2960-24TT) — puertos: Fa0/1...Fa0/24, Gi0/1, Gi0/2

Conexiones:
  R1[Serial0/1/0] <---> R2[Serial0/1/0]
  R1[GigabitEthernet0/0] <---> SW1[GigabitEthernet0/1]
  ...
```
Reportar puertos tal como los devuelve PT — sin abreviar, sin inventar.

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
| Clock rate | `64000` (seriales — solo si el usuario tiene cables serial en la topología) |
| DHCP no mencionado | Asumir SÍ |
| Routing no especificado | **PREGUNTAR** — no asumir |
| RIP o EIGRP pedido | Usar CLI manual via `configureIosDevice` — advertir que MCP no los genera |

⚠️ Si el dato existe aunque mal escrito → NO usar default, corregirlo con `[corregido]`

**PASO C — IR DIRECTO A EJECUCIÓN (sin confirmación en MODO EXAMEN):**

Mostrar resumen de 3 líneas antes de inyectar:
```
"Configurando: Router0 Fa0/0=10.x.x.1/24, Se2/0=10.0.0.1/30 | RIP V2 (CLI) | NAT en Router0."
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

> ⚠️ Verificar nombre EXACTO del puerto según modelo:
> - 1941/2901/2911: `GigabitEthernet0/0`, `GigabitEthernet0/1`
> - ISR4321: `GigabitEthernet0/0/0`, `GigabitEthernet0/0/1` ← diferente notación
> - Seriales (si HWIC-2T instalado): `Serial0/1/0`, `Serial0/1/1`

**GRUPO B — Dispositivos con IP estática** (en paralelo con routers):
```
⚠️ BUG CONOCIDO: configurePcIp() NO funciona en Server-PT

Para Server-PT (solo IP):
  pt_configure_ip(device="Server0", ip="x.x.x.x", mask="...", gateway="...")

Para Server-PT (IP + HTTP o DHCP pool):
  pt_configure_server_static(device="Server0", ip="x.x.x.x", mask="...", gateway="...", http_enabled=True)

Para PC-PT/Laptop-PT con IP estática:
  pt_send_raw('configurePcIp("PC0", false, "x.x.x.x", "255.x.x.x", "x.x.x.x", "8.8.8.8")')
  ← incluir siempre el 6to argumento dns
```

**Reglas de blindaje anti-fallos obligatorias:**
- `clock rate 64000` → SOLO si hay interfaces seriales físicas con HWIC-2T instalado
- STP wait → si hay Switch: incluir `time.sleep(30)` en script Python después de activar puertos
- Nombre lógico CLI: usar el nombre EXACTO que devolvió el escaneo (no abreviar)
- ISR4321 → siempre `0/0/0` y `0/0/1` NO `0/0`
- Cloud-PT → solo `Ethernet6` para conexiones Ethernet normales

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
configurePcIp("PC0", false, "0.0.0.0", "0.0.0.0", "0.0.0.0", "0.0.0.0")
configurePcIp("Laptop0", false, "0.0.0.0", "0.0.0.0", "0.0.0.0", "0.0.0.0")

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

## Protocolo ante fallo de bridge

Si `pt_bridge_status` falla o PT no responde:
```
1. Verificar que Packet Tracer esté abierto y con una topología activa
2. Verificar que el servidor MCP esté corriendo:
   cd C:\Users\Andrea\Desktop\MCP-Packet-Tracer && python -m src.packet_tracer_mcp
3. Verificar puertos libres: 39000 (MCP) y 54321 (bridge)
4. Si sigue sin conectar → re-pegar bootstrap en Builder Code Editor y hacer Run
5. pt_bridge_status → reintentar
6. Si sigue fallando → usar pt_deploy (clipboard) + ejecución manual
```

---

## Referencia VLSM rápida

Calcular de mayor a menor número de hosts:

| Hosts requeridos | Prefijo | Hosts útiles | Máscara |
|---|---|---|---|
| 1-2 | /30 | 2 | 255.255.255.252 |
| 3-6 | /29 | 6 | 255.255.255.248 |
| 7-14 | /28 | 14 | 255.255.255.240 |
| 15-30 | /27 | 30 | 255.255.255.224 |
| 31-62 | /26 | 62 | 255.255.255.192 |
| 63-126 | /25 | 126 | 255.255.255.128 |
| 127-254 | /24 | 254 | 255.255.255.0 |
| 255-510 | /23 | 510 | 255.255.254.0 |
| 511-1022 | /22 | 1022 | 255.255.252.0 |
| 1023-2046 | /21 | 2046 | 255.255.248.0 |
| 2047-4094 | /20 | 4094 | 255.255.240.0 |

---

## Plantillas CLI por protocolo

### SSH seguro
```
ip domain-name empresa.local
crypto key generate rsa
username admin secret admin123
line vty 0 4
 login local
 transport input ssh
```

### Interfaces GigabitEthernet (1941/2901/2911)
```
interface GigabitEthernet0/0
 description [descripcion]
 ip address [IP] [MASK]
 no shutdown
```

### Interfaces GigabitEthernet (ISR4321) ← notación diferente
```
interface GigabitEthernet0/0/0
 description [descripcion]
 ip address [IP] [MASK]
 no shutdown
```

### Interfaz Serial (requiere HWIC-2T instalado)
```
interface Serial0/1/0
 ip address [IP] [MASK]
 clock rate 64000
 no shutdown
```

### Router-on-a-Stick (subinterfaces)
```
interface GigabitEthernet0/0
 no shutdown
interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
interface GigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
```

### RIP V2 (CLI manual — MCP no lo genera)
```
router rip
 version 2
 network [red-directamente-conectada]
 no auto-summary
 default-information originate   ← solo en router con ruta default
```

### OSPF
```
router ospf 1
 router-id [X.X.X.X]
 network [red] [wildcard] area [N]
 passive-interface [LAN-interface]
```

### EIGRP (CLI manual — MCP no lo genera)
```
router eigrp [AS]
 no auto-summary
 network [red] [wildcard]
 passive-interface [LAN-interface]
```

### NAT Overload / PAT (CLI manual — MCP no lo genera)
```
interface [inside-interface]
 ip nat inside
interface [outside-interface]
 ip nat outside
access-list 1 permit [red-interna] [wildcard]
ip nat inside source list 1 interface [outside-interface] overload
ip route 0.0.0.0 0.0.0.0 [next-hop]
```

### NAT Estático
```
ip nat inside source static [ip-privada] [ip-publica]
```

### DHCP en Router
```
ip dhcp excluded-address [gateway-ip]
ip dhcp pool [NOMBRE]
 network [red] [mascara]
 default-router [gateway]
 dns-server 8.8.8.8
 lease 7
```

### DHCP Relay (helper-address)
```
interface [LAN-interface]
 ip helper-address [IP-servidor-DHCP]
```

### VLANs en Switch (CLI manual — MCP no lo genera)
```
vlan [ID]
 name [NOMBRE]
interface [puerto-acceso]
 switchport mode access
 switchport access vlan [ID]
interface [trunk-port]
 switchport mode trunk
 switchport trunk allowed vlan [ID1],[ID2]
 switchport trunk native vlan 99
```

### Port Security
```
interface FastEthernet0/1
 switchport mode access
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security violation shutdown
```

### ACL Estándar
```
access-list [1-99] deny [red] [wildcard]
access-list [1-99] permit any
interface [iface]
 ip access-group [N] out
```

### ACL Extendida
```
access-list [100-199] permit tcp [src] [wildcard] any eq [puerto]
access-list [100-199] deny ip any any
interface [iface]
 ip access-group [N] in
```

### ACL Nombrada
```
ip access-list extended [NOMBRE]
 permit tcp any any eq 80
 permit tcp any any eq 443
 deny ip any any
interface [iface]
 ip access-group [NOMBRE] in
```

### Rutas estáticas
```
ip route [red-destino] [mascara] [siguiente-salto]
ip route 0.0.0.0 0.0.0.0 [next-hop]   ← ruta default
```

### IP de management en Switch
```
interface vlan 1
 ip address [IP] [MASK]
 no shutdown
ip default-gateway [gateway]
```

---

## Comandos de diagnóstico esenciales

```
ping <ip>                    → conectividad básica
traceroute <ip>              → ruta de paquetes
show ip interface brief      → resumen interfaces e IPs
show running-config          → config actual
show startup-config          → config guardada
show vlan brief              → VLANs en switch
show interfaces trunk        → puertos trunk activos
show ip route                → tabla de enrutamiento
show ip protocols            → protocolos activos
show ip ospf neighbor        → vecinos OSPF
show ip eigrp neighbors      → vecinos EIGRP
show ip nat translations     → traducciones NAT
show ip dhcp binding         → IPs asignadas
show ip dhcp pool            → estado pools DHCP
show ip dhcp conflict        → conflictos DHCP
show mac address-table       → tabla MAC del switch
show cdp neighbors           → vecinos Cisco conectados
show port-security interface [iface]  → estado port security
copy running-config startup-config    → guardar config
```

---

## Orden recomendado para armar una red desde cero

1. Dibujar topología física (conectar cables)
2. Configurar hostnames, contraseñas y seguridad básica en todos los dispositivos
3. Crear VLANs en switches y asignar puertos de acceso
4. Configurar puertos trunk entre switches y hacia el router
5. Configurar interfaces IP en routers (subinterfaces si hay VLANs)
6. Configurar DHCP en routers o servidores
7. Configurar enrutamiento (estático u OSPF / RIP-CLI / EIGRP-CLI)
8. Probar conectividad entre todas las redes con ping
9. Configurar NAT si hay salida a Internet
10. Configurar servicios en servidores (DNS, HTTP, FTP, Email)
11. Aplicar ACLs y port-security al final
12. Guardar: `copy running-config startup-config`

---

## Reglas clave (nunca omitir)

1. **Router del usuario = `Router-PT`** — lo coloca y configura manualmente en PT
2. **Switch del usuario = `2960-24TT`**
3. **Serial entre routers = `clock rate 64000` SIEMPRE** — el usuario usa seriales por defecto
4. **Puertos del Router-PT: NUNCA asumir** — leer siempre del escaneo (el usuario agrega módulos a mano)
5. **Flujo estándar: escanear primero, reportar con puertos exactos, luego configurar**
6. ISR4321 → puertos `GigabitEthernet0/0/0` y `0/0/1`, NO `0/0` y `0/1`
7. STP 30s — esperar antes de pedir DHCP cuando hay Switches
8. RIP y EIGRP → SOLO CLI manual via `configureIosDevice`, el generador los ignorará
9. NAT / ACL / VLANs → SOLO CLI manual via `configureIosDevice`
10. `configurePcIp()` → ❌ NO funciona en Server-PT, usar `pt_configure_server_static`
11. `configurePcIp()` con IP estática → incluir siempre el 6to argumento (dns)
12. `pt_query_links` — tool oficial para escanear enlaces (limpia saltos de línea internamente). Preferir sobre bucle JS manual. Fallback al bucle JS solo si da timeout.
13. NUNCA spawning de agentes — solo MCP directo
14. NUNCA inventar datos — preguntar si falta algo crítico
15. NUNCA usar tools que no existen en el MCP (hay exactamente 26)
16. Verificar `pt_bridge_status` ANTES de cualquier operación de despliegue
