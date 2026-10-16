# JR-4 — Inventario de identificadores de marca "RustDesk"

Documento de inventario, **sin cambios de código**. Sirve para planificar
JR-4 (identidad Jali Remote) en commits pequeños, tal como exigen
`docs/JALI_REMOTE_PLAN.md` y el acta del proyecto. JR-4 sigue bloqueado
hasta cerrar JR-0 a JR-3.

Base analizada: `e5bc204` (upstream) + docs de `jali-remote/foundation`.

## Resumen

El nombre "RustDesk" se usa de tres formas muy distintas:

1. **Texto visible** (títulos, recursos del `.exe`, iconos): seguro de
   cambiar.
2. **Identidad en tiempo de ejecución** (`APP_NAME`): decide carpetas de
   configuración, nombre del servicio de Windows, protocolo `rustdesk://`,
   clase de ventana, y además **activa el "modo cliente personalizado"**
   cuando deja de valer exactamente `RustDesk`. Cambiarlo es un cambio de
   comportamiento, no solo cosmético.
3. **Protocolo e infraestructura** (puertos, mensajes, servidores, firmas):
   **no** se debe tocar en JR-4.

## 1. Seguro de cambiar (solo apariencia)

| Qué | Dónde | Notas |
| --- | --- | --- |
| Recursos de versión del runner Flutter (`ProductName`, `FileDescription`, `CompanyName`, `LegalCopyright`) | `flutter/windows/runner/Runner.rc` (líneas ~92–98) | Lo que muestra Windows en *Propiedades → Detalles*. Mantener el aviso de copyright original junto al nuevo (AGPL). |
| Recursos de versión de la librería Rust | `Cargo.toml`, `[package.metadata.winres]` | Mismo criterio. |
| Icono de la aplicación Windows | `flutter/windows/runner/resources/app_icon.ico` | Reemplazo de archivo. |
| Iconos y logos generales | `res/icon.ico`, `res/icon.png`, `res/32x32.png`, `res/64x64.png`, `res/128x128*.png`, `res/logo.svg`, `res/logo-header.svg`, `res/scalable.svg` | `res/gen_icon.sh` regenera tamaños. Los de macOS/Linux no aplican (fuera de alcance). |
| Descripción del paquete Flutter | `flutter/pubspec.yaml` (`description`) | Cosmético. **No** cambiar `name: flutter_hbb`: lo usan imports Dart en todo `flutter/lib`. |

Textos de interfaz que dicen "RustDesk" en `src/lang/*.rs`: la mayoría ya
se generan con el nombre de la app; revisar al final, no sustituir en masa
(la guía de agentes prohíbe reemplazos globales).

## 2. Identidad en tiempo de ejecución — cambiar con cuidado

`APP_NAME` vale `"RustDesk"` por defecto en
`libs/hbb_common/src/config.rs:72` (submódulo compartido con el servidor:
**no editar** ahí). Upstream solo lo cambia vía
`read_custom_client` (`src/common.rs`), que exige una configuración
**firmada con la clave privada de RustDesk** → no sirve para nosotros.

Camino recomendado para JR-4: asignar `APP_NAME` muy al principio del
arranque, en código del cliente (`src/`), en un único commit y con su propia
prueba. Efectos que hay que validar en ese commit:

| Efecto de cambiar `APP_NAME` | Dónde | Consecuencia |
| --- | --- | --- |
| Carpeta de configuración y logs | `libs/hbb_common/src/config.rs` (`ProjectDirs`, rutas de log) | `%AppData%\RustDesk` → `%AppData%\<nuevo>`. Una instalación existente **pierde** su ID, contraseña permanente y servidor configurado salvo que se migren. |
| Nombre del servicio de Windows | `src/platform/windows.rs` (`service_dispatcher::start(get_app_name())`, instalación) | El servicio pasa a llamarse distinto; desinstalar antes la versión vieja. |
| Protocolo de enlaces | `src/common.rs`, `get_uri_prefix()` | `rustdesk://` → `<nuevo>://`. |
| Clase/título de ventana para instancia única | `flutter/windows/runner/main.cpp:83` (lee `get_rustdesk_app_name` de la DLL) | Se adapta solo si la DLL devuelve el nuevo nombre. |
| **Modo cliente personalizado** | `src/common.rs`, `is_custom_client()` = `get_app_name() != "RustDesk"` | Se activa automáticamente. Efectos conocidos: desactiva la comprobación de actualizaciones al inicio (`check_software_update`), oculta la opción "Comprobar actualizaciones" en ajustes, cambia rutas del actualizador/MSI (`src/updater.rs`, `src/platform/windows.rs` ~1512, ~2083, ~2216, ~3765, ~3879), idioma por defecto (`src/lang.rs:253`) y varios elementos de la home de escritorio (`flutter/lib/desktop/pages/desktop_home_page.dart`). **Esto es deseable** para Jali Remote (no ofrecer actualizar a RustDesk oficial), pero debe probarse explícitamente. |

No renombrar en JR-4:

- Nombre del binario `rustdesk.exe` (`flutter/windows/CMakeLists.txt`,
  `BINARY_NAME`) y de la librería `librustdesk`: los usan `build.py`, los
  workflows y el empaquetado. Si se quiere otro nombre visible, que lo haga
  el instalador (acceso directo "Jali Remote"), no el build.
- `ORG = "com.carriez"` (`libs/hbb_common/src/config.rs:57`): forma parte de
  las rutas en otras plataformas; sin beneficio en Windows.

## 3. No tocar (protocolo / infraestructura / seguridad)

| Qué | Dónde | Motivo |
| --- | --- | --- |
| Servidor público por defecto `rs-ny.rustdesk.com` y su clave `RS_PUB_KEY` | `libs/hbb_common/src/config.rs:117–118` | Submódulo. Nuestro servidor se configura por opciones (JR-1), no editando el default. |
| Puertos 21115–21119 | `libs/hbb_common/src/config.rs:120–123` | Compatibilidad con `hbbs`/`hbbr`. |
| Detección de dominios `rustdesk.com` | `src/common.rs` (`is_public`) | Lógica de seguridad/TLS, no marca. |
| Claves públicas que verifican configuraciones firmadas | `src/custom_server.rs`, `src/common.rs` (`read_custom_client`) | Seguridad; cambiarlas no nos da nada sin la clave privada. |
| `admin.rustdesk.com` como API por defecto | `src/common.rs:1162` | Solo se usa con cuentas/API; el servidor API queda vacío en nuestra configuración. |
| URLs del actualizador `github.com/rustdesk/...` | `src/updater.rs` | Ver sección 4. |
| Avisos de licencia y copyright | `LICENCE`, cabeceras, "Acerca de" | Obligatorio por AGPL v3. |

## 4. Actualizaciones: riesgo para equipos instalados

- La **actualización automática** (`allow-auto-update`) está **desactivada
  por defecto**: `option2bool` exige `"Y"` para opciones `allow-*`
  (`libs/hbb_common/src/config.rs`, `option2bool`). No activarla en equipos
  de clientes: instalaría el RustDesk oficial encima de nuestra build.
- La **comprobación** de versión al inicio está activa por defecto mientras
  `APP_NAME == "RustDesk"`; solo muestra un aviso. Al cambiar `APP_NAME`
  (sección 2) desaparece sola.

## 5. Configuración del servidor sin recompilar (útil para el instalador)

Mecanismos heredados que ya permiten fijar nuestro `hbbs` sin tocar código:

- **Nombre del ejecutable**: `src/custom_server.rs`,
  `get_custom_server_from_string`, leído al arrancar desde el nombre del
  `.exe` (`src/platform/windows.rs`, `get_license_from_exe_name`). Formato:
  `rustdesk-host=<HBBS_HOST>,key=<CLAVE_PUBLICA>,.exe`. Al instalar desde
  ese `.exe`, la instalación copia host y clave a la configuración
  (`src/platform/windows.rs`, `get_license`). Limitación: la clave pública
  en base64 puede contener `/`, que Windows no admite en nombres de archivo.
  Alternativa sin esa limitación, también aceptada por el mismo código
  (`get_custom_server_from_config_string`, sin firma): el JSON
  `{"host":"<HBBS_HOST>","key":"<CLAVE_PUBLICA>","relay":"","api":""}`
  codificado en base64 URL-safe y con la cadena **invertida**, usado como
  `rustdesk--<cadena>.exe` o con `rustdesk.exe --config <cadena>` (instalado
  y como administrador).
- **Línea de comandos como administrador**, con RustDesk instalado:
  `rustdesk.exe --option custom-rendezvous-server <HBBS_HOST>` y
  `rustdesk.exe --option key <CLAVE_PUBLICA>` (`src/core_main.rs`,
  `--option`). Útil en un script post-instalación.

## Orden sugerido de commits para JR-4 (cuando se desbloquee)

1. Iconos (`app_icon.ico`, `res/icon.*`). Solo archivos binarios.
2. Recursos de versión (`Runner.rc`, `[package.metadata.winres]`),
   conservando el copyright original.
3. `APP_NAME = "Jali Remote"` en el arranque del cliente + validación de la
   tabla de la sección 2 (carpetas, servicio, modo cliente personalizado,
   reinstalación limpia).
4. Revisión de textos visibles restantes, uno a uno.

Cada commit: compilar con `python build.py --portable --flutter --skip-portable-pack --hwcodec --vram`
y repetir la prueba visual de JR-0 y una sesión de JR-1.
