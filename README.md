# CDT · Centro de Desarrollo de Talento

Aplicación web de una sola página para gestionar la capacitación y el desarrollo del personal de una organización: empleados, material de capacitación, evaluaciones, acreditaciones por antigüedad y solicitudes internas, todo con control de acceso por rol.

Está contenida en un único archivo HTML (`CDT___Centro_de_Desarrollo_de_Talento.html`), sin dependencias externas ni paso de compilación.

---

## Características

- **Inicio de sesión** con clave de usuario (o nombre completo) y contraseña.
- **Resumen** con métricas según el rol: personas activas, bajas, acreditadas, aptas por antigüedad, promedio de evaluaciones y distribución por área.
- **Empleados**: alta, edición, baja (con motivo) y reactivación; catálogos editables de áreas y puestos.
- **Material de capacitación**: texto, imágenes o PDF asignados a personas específicas, con visualización protegida (marca de agua con el nombre del usuario, sin descarga ni menú contextual).
- **Evaluaciones**: creación, asignación, puntaje mínimo opcional y registro de resultados (0–100).
- **Acreditaciones**: elegibilidad tras 1 año de antigüedad; el superior inmediato autoriza y registra el puntaje (mínimo aprobatorio: **80**).
- **Solicitudes**: restablecimiento de contraseña y solicitudes de capacitación, enviadas al superior inmediato.
- **Historial** de acciones (últimos 150 registros guardados, 300 visibles).
- **Interfaz responsiva** con menú lateral, modales y soporte para `prefers-reduced-motion`.

## Roles y permisos

| Rol | Clave | Alcance |
| --- | --- | --- |
| Dirección General | `DG` | Acceso total, incluida la zona de riesgo (borrado de datos) |
| Gerente General | `GG` | Gestiona empleados, material y evaluaciones de toda la organización |
| Gerente de Área | `GA` | Ve y gestiona a las personas de su área; autoriza acreditaciones de sus subordinados directos |
| Jefe de Área | `JA` | Visibilidad limitada a su área |
| Personal | `PE` | Solo ve su propia información, materiales y evaluaciones asignadas |

Las acciones sensibles realizadas por `DG`, `GG` y `GA` piden confirmar la contraseña.

## Uso

1. Abre el archivo `CDT___Centro_de_Desarrollo_de_Talento.html` en un navegador moderno.
2. Inicia sesión con la cuenta inicial:

   | Campo | Valor |
   | --- | --- |
   | Usuario | `DG001` |
   | Contraseña | `admin123` |

3. **Cambia la contraseña de inmediato** desde *Empleados → Editar*.
4. Define tus áreas y puestos en *Empleados → Áreas y puestos* y da de alta al resto del personal.

## Almacenamiento de datos

La aplicación elige el modo de almacenamiento al iniciar:

| Modo | Cuándo se activa | Comportamiento |
| --- | --- | --- |
| **Compartido** | Si el entorno de publicación ofrece la base de datos `claude.use('db')` | Datos sincronizados en tiempo real entre usuarios. Los materiales se guardan en una colección aparte (`mats`) y el resto en el documento `cdt/core`. Requiere permiso de colaborador para escribir. |
| **Local** | Si no hay base de datos disponible | Datos en `localStorage` del navegador (clave `cdt_v1`). **No se comparten** entre usuarios ni dispositivos; la pantalla de acceso muestra el aviso «Modo local». |

## Estructura de los datos

```text
S = {
  users:  [{ id, nombre, apellido, pass, area, puesto, ingreso, rol, superior, activo, baja }],
  areas:  [string],
  puestos:[string],
  mats:   [{ id, titulo, texto, file, mime, asig, by, fecha }],
  evals:  [{ id, titulo, desc, min, asig, res, by, fecha }],
  reqs:   [{ id, tipo: 'pass' | 'cap', de, para, titulo, texto, estado, fecha }],
  acc:    { [userId]: { score, ok, fecha, by, arch } },
  log:    [{ t, q, a, txt }]
}
```

## Reglas de negocio

- **Antigüedad para acreditación**: 365 días desde la fecha de ingreso.
- **Puntaje mínimo de acreditación**: 80 (constante `ACC_MIN`).
- **Archivos de material**: imagen o PDF, máximo 150 KB.
- **ID de usuario**: solo letras, números, `.`, `_` y `-`; debe ser único.
- **Restablecer contraseña**: la solicitud llega al superior inmediato o, si no existe o está inactivo, a la Dirección General.
- **Dar de baja** revoca el acceso de inmediato, incluso si la persona tiene sesión abierta.
- **Borrar todos los datos** (solo `DG`): exige escribir `BORRAR` y confirmar contraseña; conserva únicamente la cuenta de quien lo ejecuta.

## Personalización

| Qué | Dónde |
| --- | --- |
| Puntaje mínimo de acreditación | Constante `ACC_MIN` |
| Clave de `localStorage` | Constante `K` |
| Nombres de los roles | Objeto `RN` |
| Datos iniciales (áreas, puestos, cuenta DG) | Función `seed()` |
| Colores y tipografía | Variables CSS en `:root` |
| Antigüedad requerida | Función `elig()` |

## Consideraciones de seguridad

Es un prototipo funcional que se ejecuta íntegramente en el navegador, por lo que conviene tener presente:

- Las contraseñas se almacenan **en texto plano** y la validación ocurre del lado del cliente.
- Los permisos por rol se aplican en la interfaz, no en un servidor.
- La protección del material (marca de agua, sin descarga) disuade, pero no impide capturas de pantalla ni la inspección del navegador.
- La cuenta inicial `DG001 / admin123` es pública en el código.

Para un uso en producción con datos reales se recomienda añadir autenticación y autorización en servidor, almacenar contraseñas con hash y mover los archivos a un almacenamiento seguro.

## Tecnología

HTML, CSS y JavaScript puros (sin frameworks). Interfaz en español (`es-MX`).
