# JR-1 — Guía operativa: `hbbs` propio y sesión directa (P2P)

Issue: Jalidev-organization/rustdesk#8 (sustituye el enfoque "servidor en la PC
de Héctor" de #3 por un `hbbs` en Hetzner usado **solo** para rendezvous).

Estado de partida (comentario del issue #8, 2026-10-05):

- `hbbs` 1.1.16 instalado en el host Hetzner bajo `/opt/jali-remote`, usuario
  sin login `jali-remote`, servicio `jali-remote-hbbs.service` habilitado al
  arranque, claves en `/opt/jali-remote/data`.
- `hbbr` **no** instalado; 21117 cerrado.
- Verificación local OK para 21115/TCP, 21116/TCP y 21116/UDP.
- **Bloqueo actual:** el firewall perimetral de Hetzner Cloud descarta el
  tráfico antes de llegar al servidor.

Esta guía empieza en ese punto. No requiere cambios de código: todo lo que
sigue usa el comportamiento heredado de RustDesk, verificado en el código de
este fork (referencias al final).

> Nunca publiques en Git/GitHub: IP pública del servidor o de los clientes,
> IDs de dispositivo, contraseñas, la clave **privada** `id_ed25519`, ni
> capturas donde aparezcan. La clave **pública** no es secreta, pero tampoco
> hace falta publicarla.

---

## Paso 1 — Abrir el firewall de Hetzner Cloud

Consola Hetzner Cloud → *Firewalls* → el firewall asignado al servidor →
*Inbound rules*. Añadir exactamente:

| Protocolo | Puerto | Origen | Para qué |
| --- | --- | --- | --- |
| TCP | 21116 | `0.0.0.0/0` (y `::/0` solo si usarás IPv6) | Registro, conexión y hole punching |
| UDP | 21116 | `0.0.0.0/0` (y `::/0` solo si usarás IPv6) | Registro / heartbeat de clientes y punch UDP |
| TCP | 21115 | `0.0.0.0/0` (y `::/0` solo si usarás IPv6) | Prueba de tipo de NAT (el cliente usa ID-server − 1) |

**No** abrir: 21117 (relay, fuera de alcance), 21118/21119 (cliente web),
ni ampliar SSH. Comprobar que el firewall está **aplicado** al servidor
(pestaña *Resources* del firewall), no solo creado.

Si además hay `ufw`/`nftables` en el propio servidor, deben permitir los
mismos tres puertos (la verificación local ya pasó, así que normalmente sí).

## Paso 2 — Verificar alcance externo

Desde **una PC Windows fuera de Hetzner** (PowerShell; sustituye
`<HBBS_HOST>` por el nombre o IP del servidor, sin anotarlo en Git):

```powershell
Test-NetConnection <HBBS_HOST> -Port 21116   # Esperado: TcpTestSucceeded : True
Test-NetConnection <HBBS_HOST> -Port 21115   # Esperado: TcpTestSucceeded : True
Test-NetConnection <HBBS_HOST> -Port 21117   # Esperado: False (debe seguir cerrado)
```

`Test-NetConnection` no prueba UDP. Para 21116/UDP, en el servidor:

```bash
sudo tcpdump -ni any udp port 21116
```

y luego, en el Paso 4, al guardar la configuración en un cliente deben verse
paquetes UDP entrantes desde la IP pública de ese cliente. Si llegan TCP pero
nunca UDP, la regla UDP de Hetzner falta o no está aplicada.

Estado del servicio y clave pública (en el servidor):

```bash
systemctl status jali-remote-hbbs.service --no-pager
journalctl -u jali-remote-hbbs.service -n 50 --no-pager
sudo cat /opt/jali-remote/data/id_ed25519.pub   # esta es la "Key" de los clientes
```

Haz una copia de seguridad cifrada, fuera del servidor y fuera de Git, de
`id_ed25519` **e** `id_ed25519.pub`. Si se pierden, todos los clientes
tendrán que reconfigurarse con una clave nueva.

## Paso 3 — Comprobar que el servidor exige la clave

`hbbs` debe rechazar clientes con una clave distinta. Prueba una vez:
configura un cliente con el ID Server correcto y una **Key falsa** (por
ejemplo, la real con un carácter cambiado) e intenta conectar al otro equipo.
Resultado esperado: error **"La clave no coincide"** (*Key mismatch*). Si conecta igualmente, el
servidor no está imponiendo la clave: revisa los argumentos de
`ExecStart` del servicio y `hbbs --help` de la versión instalada antes de
seguir. Restaura después la Key correcta.

## Paso 4 — Configurar los dos clientes Windows

En **ambos** equipos (PC A = controlador, PC B = remoto), con el mismo
build verificado de JR-0 si es posible:

1. Ajustes → **Red** → **Desbloquear Ajustes de Red** (requiere elevación) → **Servidor ID/Relay**.
2. Rellenar:
   - **Servidor de IDs:** `<HBBS_HOST>` (el puerto 21116 se añade solo).
   - **Servidor Relay:** *vacío*.
   - **Servidor API:** *vacío*.
   - **Key / Clave:** el contenido de `id_ed25519.pub`.
3. Aceptar. No tocar códecs, calidad, captura ni ninguna otra opción.
4. La barra de estado de la ventana principal debe pasar a **"Listo"**.
   Si queda en "Conectando a la red de RustDesk…" o no está listo, revisa el
   Paso 2.
5. Si el equipo tiene RustDesk **instalado como servicio**, el servicio
   recoge la configuración al guardarla; si no aparece "Listo" en ~30 s,
   reinicia el servicio o la PC una vez.

Por qué "Servidor Relay vacío" sí significa "sin relay": cuando no hay relay
configurado ni lo indica el servidor, el cliente prueba `<HBBS_HOST>:21117`
como relay de último recurso (`src/rendezvous_mediator.rs`,
`get_relay_server`). Como 21117 está cerrado, ese intento **falla limpio**
en vez de desviar el tráfico. Si en algún momento se abre 21117 o se
instala `hbbr`, esta garantía desaparece.

Nota sobre STUN: el transporte WebRTC heredado usa servidores STUN públicos
(Cloudflare, Google, etc.; `libs/hbb_common/src/webrtc.rs`) solo para
descubrir direcciones. No transportan la sesión ni son `rustdesk.com`; se
anota como dependencia externa conocida, sin cambiarla en JR-1.

## Paso 5 — Confirmar que no se usa `rs-ny.rustdesk.com`

Con ambos clientes en "Listo", en cada PC Windows:

```powershell
Get-NetTCPConnection -State Established |
  Where-Object { $_.RemotePort -in 21115,21116,21117 } |
  Select-Object LocalAddress,RemoteAddress,RemotePort,OwningProcess
```

Todas las conexiones a esos puertos deben ir a la IP de `<HBBS_HOST>`.
Cualquier conexión a otra IP en 21116/21117 indica que algún proceso sigue
usando la configuración antigua (p. ej. un servicio no reiniciado).

## Paso 6 — Sesión entre redes distintas y prueba P2P

Preparación: PC B en su red habitual; PC A (portátil) en el **hotspot móvil**
que ya dio una sesión usable en JR-0B. Anota hora de inicio.

1. Desde PC A, introducir el ID de PC B y conectar; cronometrar desde
   "Conectar" hasta ver la pantalla remota.
2. En la ventana de sesión, pasar el ratón por el **icono de escudo/conexión**
   de la barra superior. Debe decir **"Conexión directa y cifrada"**
   (*Direct and encrypted connection*), seguido del transporte entre
   paréntesis: `(TCP)`, `(UDP)`, `(IPv6)` o `(WebRTC)`.
   - "Conexión retransmitida…" (*Relayed…*) = **no** es P2P → la prueba
     falla, aunque la sesión funcione.
3. Evidencia en el log del **controlador** (PC A),
   `%AppData%\RustDesk\log\`:

   ```powershell
   Select-String -Path "$env:APPDATA\RustDesk\log\*.log" `
     -Pattern 'used to establish','Failed to connect to relay','Key mismatch','Failed to make direct connection'
   ```

   Esperado, por ejemplo: `... used to establish TCP connection with ... punch`
   o `... used to establish UDP connection ...`. Si aparece `Relay` como
   transporte o `Failed to make direct connection to remote desktop`, el
   punch falló.
4. En el servidor, durante la sesión, el tráfico hacia `hbbs` debe ser
   casi nulo (solo heartbeats):

   ```bash
   sudo tcpdump -ni any 'port 21116 or port 21115' -c 200
   ```

   Mientras mueves ventanas en PC B no debe haber un flujo continuo de
   paquetes grandes en el servidor.

Después, con la misma sesión:

| # | Prueba | Cómo |
| --- | --- | --- |
| 1 | Registro | Ambos en "Listo" con el ID Server propio |
| 2 | Sin servidores públicos | Paso 5 limpio en ambos equipos |
| 3 | Pantalla | Imagen estable de PC B |
| 4 | Ratón y teclado | Abrir Bloc de notas en PC B y escribir |
| 5 | Portapapeles | Copiar texto A→B y B→A |
| 6 | Transferencia de archivos | Enviar un `.txt` inofensivo en ambos sentidos |
| 7 | Desconectar/reconectar | Cerrar la sesión y volver a conectar; anotar tiempo |
| 8 | Latencia subjetiva | Arrastrar una ventana y escribir; describir el retardo |
| 9 | Directa | Icono = "directa" + línea `used to establish` sin `Relay` |
| 10 | Sin relay | 21117 cerrado (Paso 2) y sin `Failed to connect to relay` en una sesión exitosa |

Repetir la conexión **3 veces** para no confundir un éxito aislado con un
comportamiento estable.

## Paso 7 — Si la sesión directa falla

No abrir 21117 ni instalar `hbbr` para "arreglarlo": según el issue #8,
eso es una decisión aparte.

1. Anotar el texto exacto del error y el tipo de red de cada lado (fibra,
   hotspot 4G/5G, red corporativa…).
2. En los logs, buscar las líneas de punch:

   ```powershell
   Select-String -Path "$env:APPDATA\RustDesk\log\*.log" -Pattern 'Hole Punched','peer address','used to establish','direct connection'
   ```

   Un NAT **simétrico** en ambos lados (típico de CGNAT móvil) hace muy
   improbable el punch directo; con NAT simétrico el cliente solo concede
   ~1 s al intento directo (`src/client.rs`, `connect`).
3. Probar **otra** combinación de redes (p. ej. hotspot de otra operadora o
   fibra en ambos lados) para separar "red concreta" de "fallo general".
4. Abrir un issue nuevo y acotado con esa evidencia y una de las opciones del
   issue #8: relay cercano a Perú, overlay privado (Tailscale/WireGuard) para
   esos sitios, o relay lejano solo como último recurso.

## Paso 8 — Registrar resultados (plantilla)

Pegar como comentario en el issue #8, **sin** IPs, IDs, contraseñas ni
capturas con credenciales:

```markdown
### JR-1 resultado — <fecha/hora, zona horaria>

- Build cliente: <ruta o SHA256 del runner de JR-0> (PC A) / <ídem> (PC B)
- Windows: PC A <versión> · PC B <versión>
- Redes: PC A <tipo, operadora a grandes rasgos> · PC B <tipo, operadora>
- hbbs: 1.1.16, Hetzner, sin hbbr; 21117 cerrado verificado: <sí/no>
- Firewall Hetzner: TCP 21115, TCP 21116, UDP 21116 abiertos: <sí/no>
- Key exigida (prueba "Key mismatch"): <sí/no>

| # | Prueba | Resultado | Notas |
|---|---|---|---|
| 1 | Registro en hbbs propio | | |
| 2 | Sin rs-ny.rustdesk.com | | |
| 3 | Pantalla | | |
| 4 | Ratón/teclado | | |
| 5 | Portapapeles A→B / B→A | | |
| 6 | Archivos A→B / B→A | | |
| 7 | Reconexión (tiempo) | | |
| 8 | Latencia subjetiva | | |
| 9 | Directa (texto del icono + transporte) | | |
| 10 | Sin relay | | |

- Tiempo de conexión (3 intentos): <s>, <s>, <s>
- Línea de log `used to establish ...` (sin IPs): <...>
- Problemas observados: <...>
- Decisión: <cerrar JR-1 / abrir issue de fallback>
```

## Criterio de salida (del issue #8)

Ambos clientes Windows usan el `hbbs` propio para rendezvous y establecen una
sesión remota **directa (P2P)** entre redes distintas, sin depender de
`rs-ny.rustdesk.com`. Con eso, JR-1 se puede cerrar y JR-2 (#4) queda
desbloqueado; #3 puede cerrarse como sustituido por #8.

## Referencias en el código (comportamiento heredado, sin cambios)

- Puertos: `libs/hbb_common/src/config.rs` (`RENDEZVOUS_PORT = 21116`,
  `RELAY_PORT = 21117`); prueba de NAT en ID-server − 1:
  `src/common.rs` (`increase_port(&server1, -1)`).
- Relay por defecto en el equipo remoto cuando "Relay Server" está vacío:
  `src/rendezvous_mediator.rs`, `get_relay_server`.
- Elección directa vs relay, timeouts y mensajes de log
  (`used to establish`, `Failed to make direct connection to remote desktop`,
  `Key mismatch`): `src/client.rs`.
- Texto del indicador de conexión: `flutter/lib/common.dart`,
  `getConnectionText`.
- STUN por defecto de WebRTC: `libs/hbb_common/src/webrtc.rs`.
