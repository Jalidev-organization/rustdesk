# Plantillas de validación — JR-0B (#7) y JR-2 (#4)

Plantillas para registrar pruebas reales Windows → Windows y pegarlas como
comentario en el issue correspondiente. La plantilla de JR-1 (#8) está en
`docs/tasks/JR-1-SELF-HOSTED-HBBS.md`, Paso 8.

## Reglas de privacidad (todas las plantillas)

**No** escribir nunca en GitHub: IPs públicas, IDs de dispositivo,
contraseñas (temporales o permanentes), claves privadas, nombres de usuario
de Windows de terceros, ni capturas donde se vea cualquiera de esos datos.
Para redes, usar solo descripciones generales: "fibra hogar", "hotspot 4G
operadora X", "red de oficina".

Resultados: `PASA` · `FALLA` · `PARCIAL` · `NO PROBADO` (con motivo).
Cualquier `FALLA` reproducible se convierte en un issue nuevo y acotado; no
se corrige dentro de la tarea de validación.

## Cómo obtener datos útiles sin exponer secretos

| Dato | Cómo obtenerlo |
| --- | --- |
| Versión de Windows | `winver` o `(Get-ComputerInfo).OsName, (Get-ComputerInfo).OsBuildNumber` |
| Hash del cliente | `Get-FileHash <ruta>\rustdesk.exe -Algorithm SHA256` |
| Directa o retransmitida | Pasar el ratón sobre el icono de escudo de la sesión: "Conexión directa y cifrada" / "Conexión retransmitida y cifrada" y el transporte entre paréntesis |
| Log del cliente (portátil) | `%AppData%\RustDesk\log\` |
| Log del servicio (instalado) | `C:\Windows\ServiceProfiles\LocalService\AppData\Roaming\RustDesk\log\` |
| Línea de conexión en el log | `Select-String -Path <carpeta>\*.log -Pattern 'used to establish'` |

Antes de pegar un fragmento de log, buscar y borrar IPs (`\d+\.\d+\.\d+\.\d+`)
e IDs.

---

## Plantilla JR-0B — Sesión base entre redes (#7)

Condiciones: build de JR-0 en ambos equipos si es posible; **sin** servidor
propio (infraestructura heredada); sin acceso desatendido.

```markdown
### JR-0B resultado — <fecha/hora, zona horaria>

- PC A (controlador): Windows <versión/build> · build <SHA256 o versión oficial>
- PC B (remoto):      Windows <versión/build> · build <SHA256 o versión oficial>
- Red PC A: <tipo general> · Red PC B: <tipo general>
- Servidor: público heredado (sin servidor propio)

| # | Prueba | Resultado | Notas |
|---|---|---|---|
| 1 | Conexión establecida; tiempo hasta ver pantalla (s) | | |
| 2 | Pantalla remota visible y estable | | |
| 3 | Ratón | | |
| 4 | Teclado (incl. acentos y ñ) | | |
| 5 | Pantalla completa | | |
| 6 | Portapapeles texto A→B | | |
| 7 | Portapapeles texto B→A | | |
| 8 | Transferencia de un archivo inofensivo | | |
| 9 | Desconectar y reconectar | | |
| 10 | Cerrar y reabrir RustDesk en PC B, reconectar | | |
| 11 | Directa / retransmitida (texto del icono) | | |
| 12 | Calidad: latencia, imagen, tirones, cortes | | |

- Errores exactos (sin IPs/IDs): <...>
- Observaciones: <...>
- Conclusión: <base sana → seguir con JR-1 / problema → issue nuevo #...>
```

---

## Plantilla JR-2 — Matriz de soporte Windows (#4)

Requisitos previos: JR-1 cerrado (ambos equipos usando el `hbbs` propio).
PC B debe tener RustDesk **instalado** (no portátil) para las pruebas de
acceso desatendido, UAC, pantalla de bloqueo y reinicio: la versión
portátil no puede interactuar con el escritorio seguro de Windows.

Preparación del acceso desatendido en PC B (solo en equipos propios o con
autorización expresa del dueño):

1. Instalar RustDesk (botón **Instalar** de la ventana principal).
2. Ajustes → **Seguridad** → desbloquear → **Establecer contraseña
   permanente** con una contraseña larga y única. Guardarla en un gestor de
   contraseñas, nunca en GitHub.
3. Dejar **Usar ambas contraseñas** o **Usar contraseña permanente** según
   se vaya a probar.
4. Comprobar que el servicio arranca con Windows:
   `Get-Service RustDesk | Select-Object Status, StartType`.

```markdown
### JR-2 resultado — <fecha/hora, zona horaria>

- PC A: Windows <versión> · build <hash> · red <tipo>
- PC B: Windows <versión> · build <hash> · red <tipo> · instalado: <sí/no>
- Servidor de IDs: propio (JR-1) · Relay: <ninguno / ...>
- Monitores en PC B: <n>

| # | Prueba | Resultado | Notas |
|---|---|---|---|
| 1 | Pantalla, ratón y teclado | | |
| 2 | Pantalla completa "como sentado delante" | | |
| 3 | Portapapeles texto A→B / B→A | | |
| 4 | Portapapeles imagen A→B / B→A | | |
| 5 | Transferencia de archivos (ventana de archivos) | | |
| 6 | Copiar/pegar archivos entre PCs (Ctrl+C / Ctrl+V) | | |
| 7 | Arrastrar y soltar archivos A→B (describir el UX real) | | |
| 8 | Arrastrar y soltar archivos B→A (describir el UX real) | | |
| 9 | Acceso desatendido con contraseña permanente, sin nadie en PC B | | |
| 10 | UAC: instalar un programa en PC B y aceptar el aviso de UAC | | |
| 11 | Escritorio seguro (UAC atenuado) visible y controlable | | |
| 12 | Reiniciar PC B desde la sesión y reconectar sin intervención | | |
| 13 | Pantalla de inicio de sesión / bloqueo: ver y escribir contraseña | | |
| 14 | Ctrl+Alt+Supr enviado desde la barra de la sesión | | |
| 15 | Directa / retransmitida (texto del icono) | | |
| 16 | Varios monitores (solo si PC B tiene más de uno) | | |

- Tiempo de reconexión tras reinicio (s): <...>
- Errores exactos (sin IPs/IDs/contraseñas): <...>
- Funciones que faltan → issues nuevos: <#..., #...>
- Conclusión: <apto para soporte real / bloqueado por ...>
```

### Notas para las pruebas más delicadas

- **UAC (10–11):** si la pantalla se queda negra o congelada al aparecer el
  aviso, anotar si PC B tenía RustDesk instalado como servicio. Con la
  versión portátil es el comportamiento esperado, no un fallo nuevo.
- **Reinicio (12):** usar la acción de reinicio de la sesión; si no aparece,
  anotar por qué (permiso desactivado en PC B) en lugar de reiniciar a mano.
- **Arrastrar y soltar (7–8):** describir exactamente qué ocurre (nada, abre
  la ventana de archivos, copia directa…). El plan pide registrar el UX real,
  no suponer que existe.
- **Bloqueo (13):** probar tras `Win+L` en PC B y tras cerrar sesión de
  Windows.
