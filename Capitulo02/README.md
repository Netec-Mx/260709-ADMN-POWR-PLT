# Implementación de seguridad por unidades de negocio y roles en Dataverse

## Metadatos

| Campo | Detalle |
|---|---|
| **Duración estimada** | 15 minutos |
| **Complejidad** | Media |
| **Nivel Bloom** | Aplicar (Apply) |
| **Módulo** | 2 — Seguridad y Gobierno en Dataverse |
| **Práctica** | 2 de 4 |

---

## Descripción General

En esta práctica configurarás la capa de seguridad de Dataverse para la organización ficticia **Contoso Industries**, la cual cuenta con tres departamentos: Ventas, Recursos Humanos y Finanzas. Crearás unidades de negocio que reflejen esa jerarquía, definirás roles de seguridad personalizados con privilegios diferenciados sobre tablas estándar y una tabla personalizada, y asignarás usuarios de prueba a cada unidad y rol. Al finalizar, comprenderás cómo el modelo de seguridad de Dataverse —autenticación, autorización y segmentación de datos— controla de forma granular el acceso a los registros según la posición del usuario en la jerarquía organizacional.

---

## Objetivos de Aprendizaje

Al completar esta práctica, serás capaz de:

- [ ] Crear y estructurar unidades de negocio en Dataverse que representen la jerarquía departamental de Contoso Industries.
- [ ] Configurar roles de seguridad personalizados con privilegios específicos (Crear, Leer, Escribir, Eliminar) y distintos alcances sobre tablas seleccionadas.
- [ ] Asignar usuarios de prueba a unidades de negocio y roles de seguridad para validar el aislamiento de datos entre departamentos.
- [ ] Interpretar el comportamiento del acceso derivado del propietario (*Owner-derived Access*) verificando la visibilidad de registros entre usuarios de distintas unidades.
- [ ] Configurar una regla de restricción de aplicaciones para controlar qué aplicaciones pueden acceder al entorno Dataverse.

---

## Requisitos Previos

### Conocimiento previo

- Haber completado la **Práctica 1** del curso o tener acceso a un entorno Sandbox con Dataverse habilitado.
- Comprensión básica del modelo de seguridad de Dataverse: tablas, registros, privilegios y roles (cubierto en la Lección 2.1).
- Familiaridad con la navegación del Centro de Administración de Power Platform.

### Acceso y cuentas

| Requisito | Detalle |
|---|---|
| **Tenant de Microsoft 365** | Tenant de desarrollo con permisos de Administrador Global o Administrador de Power Platform |
| **Entorno Sandbox** | Entorno con Dataverse habilitado (creado en Práctica 1) |
| **Usuarios de prueba** | Mínimo 3 usuarios con licencias de Power Apps asignadas: `usuario.ventas@[tenant].onmicrosoft.com`, `usuario.rrhh@[tenant].onmicrosoft.com`, `usuario.finanzas@[tenant].onmicrosoft.com` |
| **Capacidad Dataverse** | Al menos 500 MB disponibles en el tenant |

> **⚠️ Nota para el instructor:** Los tres usuarios de prueba deben estar creados en Microsoft Entra ID y tener licencias de Power Apps (plan por usuario o incluidas en E5) asignadas antes de iniciar esta práctica. Si los usuarios no existen, consulta la sección de Requisitos Previos al final de este documento.

---

## Entorno de Laboratorio

### Hardware recomendado

| Componente | Mínimo | Recomendado |
|---|---|---|
| Procesador | Intel Core i5 8ª gen / AMD Ryzen 5 | Intel Core i7 / AMD Ryzen 7 |
| Memoria RAM | 8 GB | 16 GB |
| Resolución de pantalla | 1366 × 768 | 1920 × 1080 |
| Conexión a Internet | 10 Mbps | 25 Mbps o superior |

### Software necesario

| Herramienta | Versión | Acceso |
|---|---|---|
| Microsoft Edge o Google Chrome | 120.0 o superior | Navegador local |
| Power Platform Admin Center | SaaS | https://admin.powerplatform.microsoft.com |
| Microsoft Power Apps | SaaS | https://make.powerapps.com |
| Microsoft Entra ID (Azure AD) | SaaS | https://entra.microsoft.com |

### Verificación del entorno antes de comenzar

Abre una terminal (PowerShell o símbolo del sistema) y verifica que tienes instalado el **Power Platform CLI** si deseas usar comandos opcionales de verificación:

```powershell
pac --version
# Resultado esperado: Microsoft PowerApps CLI [versión 1.30 o superior]
```

Si el CLI no está disponible, todas las tareas pueden completarse íntegramente desde la interfaz web del Centro de Administración.

---

## Pasos del Laboratorio

---

### Paso 1: Verificar el entorno Sandbox y acceder al Centro de Administración

**Objetivo:** Confirmar que el entorno Sandbox de la Práctica 1 está disponible y acceder a la sección de seguridad de Dataverse.

#### Instrucciones

1. Abre el navegador e ingresa a **https://admin.powerplatform.microsoft.com** con tu cuenta de administrador.

2. En el panel izquierdo, selecciona **Entornos**.

3. Localiza el entorno Sandbox creado en la Práctica 1 (debe llamarse algo similar a `Contoso-Sandbox-Dev`). Confirma que su estado es **Listo** y que la columna **Tipo** indica **Sandbox**.

4. Haz clic sobre el nombre del entorno para abrir su panel de detalles.

5. Verifica que en la sección **Detalles** aparece **Base de datos de Dataverse** con el estado **Aprovisionado**.

   > Si el entorno no existe o Dataverse no está aprovisionado, detente y contacta al instructor para obtener un entorno preconfigurado.

6. Toma nota de la **URL del entorno** (formato: `https://[nombre-org].crm.dynamics.com`). La necesitarás en pasos posteriores.

#### Resultado esperado

El panel de detalles del entorno muestra estado **Listo**, tipo **Sandbox** y Dataverse aprovisionado con una URL de entorno válida.

#### Verificación

En la parte superior del panel de detalles del entorno, el indicador de estado debe mostrar un círculo verde con la etiqueta **Listo**.

---

### Paso 2: Crear las unidades de negocio para Contoso Industries

**Objetivo:** Estructurar la jerarquía organizacional de Contoso Industries creando tres unidades de negocio secundarias bajo la unidad raíz: Ventas, Recursos Humanos y Finanzas.

#### Instrucciones

1. Desde el panel de detalles del entorno Sandbox, selecciona **Configuración** en la barra de acciones superior.

2. En la página de Configuración, despliega la sección **Usuarios + permisos** y haz clic en **Unidades de negocio**.

3. Observa que ya existe una unidad de negocio raíz con el nombre de la organización (por ejemplo, `Contoso Industries`). Esta es la unidad raíz y **no debe modificarse ni eliminarse**.

4. Para crear la primera unidad de negocio, haz clic en **+ Nueva** en la barra de comandos superior.

5. Completa el formulario con los siguientes datos para la unidad de **Ventas**:

   | Campo | Valor |
   |---|---|
   | **Nombre** | `Ventas` |
   | **Unidad de negocio principal** | `[Nombre de la unidad raíz]` |
   | **División** | `Comercial` |
   | **Descripción** | `Departamento de Ventas de Contoso Industries` |

6. Haz clic en **Guardar y cerrar**.

7. Repite los pasos 4–6 para crear la unidad de **Recursos Humanos**:

   | Campo | Valor |
   |---|---|
   | **Nombre** | `Recursos Humanos` |
   | **Unidad de negocio principal** | `[Nombre de la unidad raíz]` |
   | **División** | `Operaciones` |
   | **Descripción** | `Departamento de RRHH de Contoso Industries` |

8. Repite los pasos 4–6 para crear la unidad de **Finanzas**:

   | Campo | Valor |
   |---|---|
   | **Nombre** | `Finanzas` |
   | **Unidad de negocio principal** | `[Nombre de la unidad raíz]` |
   | **División** | `Administración` |
   | **Descripción** | `Departamento de Finanzas de Contoso Industries` |

9. Una vez creadas las tres unidades, la lista de unidades de negocio debe mostrar la siguiente jerarquía:

   ```
   [Unidad Raíz — Contoso Industries]
   ├── Ventas
   ├── Recursos Humanos
   └── Finanzas
   ```

#### Resultado esperado

La lista de unidades de negocio muestra cuatro entradas: la unidad raíz más las tres unidades secundarias recién creadas (Ventas, Recursos Humanos, Finanzas), todas con estado **Activo**.

#### Verificación

Haz clic en la unidad `Ventas` y confirma que el campo **Unidad de negocio principal** apunta correctamente a la unidad raíz de Contoso Industries. Repite la verificación para `Recursos Humanos` y `Finanzas`.

---

### Paso 3: Crear la tabla personalizada "Solicitud Interna"

**Objetivo:** Crear una tabla personalizada en Dataverse sobre la cual se configurarán privilegios diferenciados en los roles de seguridad.

#### Instrucciones

1. Abre una nueva pestaña del navegador y ve a **https://make.powerapps.com**.

2. En el selector de entornos (esquina superior derecha), asegúrate de estar trabajando en el entorno Sandbox de Contoso Industries.

3. En el panel izquierdo, selecciona **Tablas** (puede estar bajo el menú **Dataverse**).

4. Haz clic en **+ Nueva tabla** → **Agregar columnas y datos**.

5. En el campo **Nombre de la tabla**, escribe: `Solicitud Interna`

   > Dataverse generará automáticamente el nombre lógico como `contoso_solicitudinterna` o similar según el prefijo del publicador.

6. Verifica que la opción **Habilitar datos adjuntos** esté desactivada para simplificar la tabla en este laboratorio.

7. Haz clic en **Crear**.

8. Una vez creada la tabla, agrega una columna adicional para enriquecer la estructura:
   - Haz clic en **+ Nueva columna**.
   - **Nombre para mostrar:** `Departamento Solicitante`
   - **Tipo de datos:** `Texto` → `Una línea de texto`
   - **Requerido:** `Opcional`
   - Haz clic en **Guardar**.

9. Agrega una segunda columna:
   - **Nombre para mostrar:** `Estado Solicitud`
   - **Tipo de datos:** `Elección` → crea las opciones: `Pendiente`, `Aprobada`, `Rechazada`
   - **Requerido:** `Opcional`
   - Haz clic en **Guardar**.

10. Haz clic en **Guardar tabla** para confirmar todos los cambios.

#### Resultado esperado

La tabla `Solicitud Interna` aparece en la lista de tablas del entorno con las columnas `Nombre`, `Departamento Solicitante` y `Estado Solicitud`. El estado de la tabla es **Activo**.

#### Verificación

En la lista de tablas, busca `Solicitud Interna`. Haz clic sobre ella y confirma que las tres columnas definidas aparecen en la pestaña **Columnas**.

---

### Paso 4: Crear roles de seguridad personalizados para cada departamento

**Objetivo:** Definir tres roles de seguridad con privilegios diferenciados sobre las tablas Cuenta, Contacto y Solicitud Interna, modelando los requisitos de acceso de cada departamento de Contoso Industries.

#### Instrucciones

1. Regresa al **Centro de Administración de Power Platform** (https://admin.powerplatform.microsoft.com).

2. Navega a **Entornos** → selecciona el entorno Sandbox → **Configuración** → **Usuarios + permisos** → **Roles de seguridad**.

3. Se abrirá la lista de roles de seguridad del entorno. Haz clic en **+ Nuevo rol** en la barra de comandos.

4. **Crear el rol para Ventas:**

   a. En el campo **Nombre del rol**, escribe: `Rol Ventas - Contoso`

   b. En el campo **Unidad de negocio**, selecciona: `Ventas`

   c. Haz clic en **Guardar** (sin cerrar) para activar las pestañas de privilegios.

   d. Selecciona la pestaña **Entidades principales** (o busca las tablas en la pestaña de personalización).

   e. Localiza la tabla **Cuenta** y configura los siguientes privilegios haciendo clic en los círculos de cada acción hasta alcanzar el alcance indicado:

   | Tabla | Crear | Leer | Escribir | Eliminar |
   |---|---|---|---|---|
   | **Cuenta** | 🔵 Unidad de negocio | 🔵 Unidad de negocio | 🔵 Unidad de negocio | ❌ Ninguno |
   | **Contacto** | 🔵 Unidad de negocio | 🔵 Unidad de negocio | 🔵 Unidad de negocio | 🟡 Usuario |
   | **Solicitud Interna** | 🟡 Usuario | 🟡 Usuario | 🟡 Usuario | 🟡 Usuario |

   > **Referencia de símbolos de alcance** (según Lección 2.1):
   > - 🟡 Usuario: solo registros propios
   > - 🔵 Unidad de negocio: registros de toda la unidad
   > - 🟢 Principal y secundarias: unidad propia y subordinadas
   > - 🔴 Organización: todos los registros
   > - ❌ Ninguno: sin acceso

   f. Haz clic en **Guardar y cerrar**.

5. **Crear el rol para Recursos Humanos:**

   a. Haz clic en **+ Nuevo rol**.

   b. **Nombre del rol:** `Rol RRHH - Contoso`

   c. **Unidad de negocio:** `Recursos Humanos`

   d. Haz clic en **Guardar** para activar las pestañas.

   e. Configura los privilegios:

   | Tabla | Crear | Leer | Escribir | Eliminar |
   |---|---|---|---|---|
   | **Cuenta** | ❌ Ninguno | 🟡 Usuario | ❌ Ninguno | ❌ Ninguno |
   | **Contacto** | 🔵 Unidad de negocio | 🔵 Unidad de negocio | 🔵 Unidad de negocio | 🔵 Unidad de negocio |
   | **Solicitud Interna** | 🔵 Unidad de negocio | 🔵 Unidad de negocio | 🔵 Unidad de negocio | ❌ Ninguno |

   f. Haz clic en **Guardar y cerrar**.

6. **Crear el rol para Finanzas:**

   a. Haz clic en **+ Nuevo rol**.

   b. **Nombre del rol:** `Rol Finanzas - Contoso`

   c. **Unidad de negocio:** `Finanzas`

   d. Haz clic en **Guardar** para activar las pestañas.

   e. Configura los privilegios:

   | Tabla | Crear | Leer | Escribir | Eliminar |
   |---|---|---|---|---|
   | **Cuenta** | ❌ Ninguno | 🟢 Principal y secundarias | ❌ Ninguno | ❌ Ninguno |
   | **Contacto** | ❌ Ninguno | 🟢 Principal y secundarias | ❌ Ninguno | ❌ Ninguno |
   | **Solicitud Interna** | 🟡 Usuario | 🟢 Principal y secundarias | 🟡 Usuario | ❌ Ninguno |

   > **Nota de diseño:** El rol de Finanzas tiene alcance "Principal y secundarias" en lectura porque necesita visibilidad sobre registros de toda la organización para procesos de auditoría, pero no puede crear ni modificar registros de otras unidades.

   f. Haz clic en **Guardar y cerrar**.

#### Resultado esperado

La lista de roles de seguridad muestra los tres nuevos roles: `Rol Ventas - Contoso`, `Rol RRHH - Contoso` y `Rol Finanzas - Contoso`, cada uno asociado a su respectiva unidad de negocio.

#### Verificación

Haz clic en `Rol Ventas - Contoso` y navega a la tabla **Cuenta**. Confirma que los privilegios de Leer y Escribir muestran el ícono de alcance **Unidad de negocio** (círculo azul) y que Eliminar aparece sin alcance (sin relleno).

---

### Paso 5: Asignar usuarios de prueba a unidades de negocio y roles

**Objetivo:** Vincular cada usuario de prueba a su unidad de negocio correspondiente y asignarle el rol de seguridad apropiado.

#### Instrucciones

1. En el Centro de Administración, navega a: **Entornos** → `[Entorno Sandbox]` → **Configuración** → **Usuarios + permisos** → **Usuarios**.

2. Si los usuarios de prueba no aparecen en la lista, haz clic en **+ Agregar usuario** y búscalos por su dirección de correo electrónico. Añádelos uno a uno al entorno.

   > Los usuarios deben tener licencias de Power Apps asignadas en Microsoft 365 para poder ser añadidos al entorno.

3. **Asignar `usuario.ventas` a la unidad Ventas:**

   a. Localiza `usuario.ventas@[tenant].onmicrosoft.com` en la lista y haz clic sobre su nombre.

   b. En el panel de detalles del usuario, busca la sección **Unidad de negocio** y haz clic en el botón de edición (ícono de lápiz).

   c. Selecciona `Ventas` en el selector de unidades de negocio y confirma el cambio.

   > **⚠️ Advertencia:** Cambiar la unidad de negocio de un usuario elimina automáticamente todos sus roles de seguridad previamente asignados. Siempre cambia la unidad de negocio **antes** de asignar roles.

   d. De vuelta en el panel del usuario, haz clic en **Administrar roles de seguridad**.

   e. En el panel lateral que se abre, marca la casilla junto a `Rol Ventas - Contoso`.

   f. Haz clic en **Guardar**.

4. **Asignar `usuario.rrhh` a la unidad Recursos Humanos:**

   a. Localiza `usuario.rrhh@[tenant].onmicrosoft.com` y abre su panel de detalles.

   b. Cambia la **Unidad de negocio** a `Recursos Humanos`.

   c. Haz clic en **Administrar roles de seguridad** y selecciona `Rol RRHH - Contoso`.

   d. Haz clic en **Guardar**.

5. **Asignar `usuario.finanzas` a la unidad Finanzas:**

   a. Localiza `usuario.finanzas@[tenant].onmicrosoft.com` y abre su panel de detalles.

   b. Cambia la **Unidad de negocio** a `Finanzas`.

   c. Haz clic en **Administrar roles de seguridad** y selecciona `Rol Finanzas - Contoso`.

   d. Haz clic en **Guardar**.

#### Resultado esperado

Los tres usuarios de prueba aparecen en la lista de usuarios del entorno, cada uno con su unidad de negocio correcta y su rol de seguridad asignado.

#### Verificación

Abre el panel de detalles de `usuario.ventas`. Confirma que:
- **Unidad de negocio:** `Ventas`
- **Roles de seguridad:** incluye `Rol Ventas - Contoso`

Repite la verificación para los otros dos usuarios.

---

### Paso 6: Explorar el acceso derivado del propietario (Owner-derived Access)

**Objetivo:** Comprender cómo el modelo de acceso derivado del propietario determina la visibilidad de registros entre usuarios de distintas unidades de negocio, creando registros de prueba con el usuario administrador y verificando la visibilidad desde los usuarios de prueba.

#### Instrucciones

1. Con tu cuenta de **administrador**, abre una nueva pestaña y navega a **https://make.powerapps.com**.

2. Asegúrate de estar en el entorno Sandbox de Contoso Industries.

3. En el panel izquierdo, selecciona **Tablas** → **Solicitud Interna**.

4. Haz clic en la pestaña **Editar** para abrir la vista de datos de la tabla.

5. Crea tres registros de prueba haciendo clic en **+ Nueva fila** para cada uno:

   | Nombre | Departamento Solicitante | Estado Solicitud |
   |---|---|---|
   | `Solicitud-VENTAS-001` | `Ventas` | `Pendiente` |
   | `Solicitud-RRHH-001` | `Recursos Humanos` | `Pendiente` |
   | `Solicitud-FINANZAS-001` | `Finanzas` | `Pendiente` |

   > **Nota:** Estos tres registros son creados por el administrador, por lo que el propietario (*owner*) de cada registro es la cuenta de administrador, que pertenece a la unidad de negocio raíz.

6. Guarda los cambios y cierra la vista de edición.

7. **Verificar visibilidad desde el rol de Ventas:**

   Abre una ventana de incógnito (Ctrl+Shift+N en Edge/Chrome) e inicia sesión con las credenciales de `usuario.ventas@[tenant].onmicrosoft.com`.

   a. Navega a **https://make.powerapps.com** y selecciona el entorno Sandbox.

   b. Ve a **Tablas** → **Solicitud Interna** → **Editar**.

   c. Observa cuántos registros son visibles para este usuario.

   > **Resultado esperado para Ventas:** El usuario de Ventas tiene alcance **Usuario** en la tabla Solicitud Interna, lo que significa que solo puede ver registros que él mismo posee. Como los tres registros fueron creados por el administrador, `usuario.ventas` **no verá ningún registro** en esta vista.

   d. Cierra la ventana de incógnito.

8. **Verificar visibilidad desde el rol de Finanzas:**

   Abre otra ventana de incógnito e inicia sesión con `usuario.finanzas@[tenant].onmicrosoft.com`.

   a. Navega a **https://make.powerapps.com** → entorno Sandbox → **Tablas** → **Solicitud Interna** → **Editar**.

   b. Observa cuántos registros son visibles.

   > **Resultado esperado para Finanzas:** El usuario de Finanzas tiene alcance **Principal y secundarias** en lectura de Solicitud Interna. Sin embargo, los registros son propiedad del administrador (unidad raíz). Dependiendo de si la unidad raíz es considerada "principal" de Finanzas, la visibilidad puede variar. En la mayoría de las configuraciones estándar, el administrador de la unidad raíz es visible para todos los alcances que incluyen la raíz. Documenta el comportamiento observado.

   c. Cierra la ventana de incógnito.

9. **Crear un registro como usuario.ventas para validar la propiedad:**

   Abre una nueva ventana de incógnito con `usuario.ventas@[tenant].onmicrosoft.com`.

   a. Navega a **Tablas** → **Solicitud Interna** → **Editar**.

   b. Haz clic en **+ Nueva fila** y crea el registro:

   | Nombre | Departamento Solicitante | Estado Solicitud |
   |---|---|---|
   | `Solicitud-VENTAS-002` | `Ventas` | `Pendiente` |

   c. Guarda el registro. Ahora `usuario.ventas` es el propietario de este registro.

   d. Verifica que `Solicitud-VENTAS-002` aparece en la lista para `usuario.ventas` (ya que el alcance Usuario permite ver registros propios).

   e. Cierra la ventana de incógnito.

10. **Verificar si usuario.rrhh puede ver Solicitud-VENTAS-002:**

    Abre una ventana de incógnito con `usuario.rrhh@[tenant].onmicrosoft.com`.

    a. Navega a **Tablas** → **Solicitud Interna** → **Editar**.

    b. Verifica si `Solicitud-VENTAS-002` es visible.

    > **Resultado esperado para RRHH:** El usuario de RRHH tiene alcance **Unidad de negocio** en Solicitud Interna. El registro `Solicitud-VENTAS-002` pertenece a `usuario.ventas`, que está en la unidad `Ventas` (diferente a `Recursos Humanos`). Por lo tanto, `usuario.rrhh` **no debería ver** ese registro, confirmando el aislamiento entre unidades de negocio.

    c. Documenta el resultado en tu cuaderno de laboratorio.

    d. Cierra la ventana de incógnito.

#### Resultado esperado

La exploración demuestra que el acceso derivado del propietario funciona correctamente: los usuarios solo ven registros según el alcance definido en su rol de seguridad y la unidad de negocio a la que pertenece el propietario del registro.

#### Verificación

Completa la siguiente tabla de verificación con los resultados observados:

| Usuario | Registros visibles en Solicitud Interna | ¿Coincide con lo esperado? |
|---|---|---|
| `usuario.ventas` (antes de crear su propio registro) | 0 | ✅ / ❌ |
| `usuario.ventas` (después de crear `Solicitud-VENTAS-002`) | 1 | ✅ / ❌ |
| `usuario.rrhh` | 0 (solo los de su unidad) | ✅ / ❌ |
| `usuario.finanzas` | Varía (documentar) | ✅ / ❌ |

---

### Paso 7: Configurar restricción de acceso de aplicaciones al entorno Dataverse

**Objetivo:** Configurar una política de restricción de aplicaciones en el entorno para controlar qué aplicaciones pueden acceder a los datos de Dataverse, implementando el principio de mínimo privilegio a nivel de aplicación.

#### Instrucciones

1. Regresa al **Centro de Administración de Power Platform** con tu cuenta de administrador.

2. Navega a **Entornos** → selecciona el entorno Sandbox → **Configuración**.

3. Despliega la sección **Producto** y selecciona **Características**.

4. Busca la sección **Seguridad de la aplicación** o **Acceso de aplicaciones al entorno**.

   > La ubicación exacta puede variar según la versión del portal. Si no encuentras esta opción en Características, busca en **Configuración** → **Privacidad + Seguridad**.

5. Localiza la opción **Restringir acceso entre entornos** o **Aplicaciones permitidas en este entorno** y actívala si está disponible en tu tenant.

6. **Configurar el registro de aplicaciones de Dataverse (método alternativo mediante Entra ID):**

   Si la opción anterior no está disponible, procede con la configuración en Microsoft Entra ID:

   a. Abre una nueva pestaña y navega a **https://entra.microsoft.com**.

   b. En el panel izquierdo, ve a **Aplicaciones** → **Registros de aplicaciones**.

   c. Busca la aplicación **Dataverse** o **Common Data Service** en la lista de aplicaciones empresariales.

   d. Selecciona **Aplicaciones empresariales** → **Todas las aplicaciones** → busca `Power Apps`.

   e. Observa la configuración de **Asignación de usuarios necesaria**. Si está en **Sí**, solo los usuarios o grupos explícitamente asignados pueden acceder a la aplicación.

7. **Documentar la configuración actual:**

   En el Centro de Administración, navega a **Entornos** → `[Entorno Sandbox]` → **Configuración** → **Usuarios + permisos** → **Roles de seguridad**.

   Verifica que el rol **Usuario básico** (Basic User) esté correctamente configurado, ya que este rol es el mínimo necesario para que cualquier usuario acceda al entorno. Haz clic sobre él y revisa sus privilegios en la pestaña **Entidades principales**.

8. **Verificar que los usuarios de prueba tienen el rol Usuario básico:**

   Navega a **Usuarios** y abre el panel de detalles de cada usuario de prueba. Confirma que además de su rol departamental, cada usuario tiene asignado el rol **Usuario básico** (este rol se asigna automáticamente al agregar usuarios al entorno, pero es importante verificarlo).

   Si algún usuario no tiene el rol **Usuario básico**:
   - Haz clic en **Administrar roles de seguridad**.
   - Marca la casilla **Usuario básico**.
   - Haz clic en **Guardar**.

9. **Registrar la configuración de seguridad aplicada:**

   Abre el Bloc de notas o cualquier editor de texto y documenta el resumen de configuración:

   ```
   ═══════════════════════════════════════════════════════
   RESUMEN DE CONFIGURACIÓN DE SEGURIDAD - CONTOSO INDUSTRIES
   Entorno: [Nombre del entorno Sandbox]
   Fecha: [Fecha actual]
   ═══════════════════════════════════════════════════════

   UNIDADES DE NEGOCIO:
   - Raíz: [Nombre de la unidad raíz]
     ├── Ventas
     ├── Recursos Humanos
     └── Finanzas

   ROLES CREADOS:
   - Rol Ventas - Contoso → Unidad: Ventas
   - Rol RRHH - Contoso → Unidad: Recursos Humanos
   - Rol Finanzas - Contoso → Unidad: Finanzas

   ASIGNACIONES DE USUARIOS:
   - usuario.ventas → Ventas → Rol Ventas - Contoso + Usuario básico
   - usuario.rrhh → Recursos Humanos → Rol RRHH - Contoso + Usuario básico
   - usuario.finanzas → Finanzas → Rol Finanzas - Contoso + Usuario básico

   TABLA PERSONALIZADA:
   - Solicitud Interna (contoso_solicitudinterna)
     Columnas: Nombre, Departamento Solicitante, Estado Solicitud
   ═══════════════════════════════════════════════════════
   ```

#### Resultado esperado

Los tres usuarios de prueba tienen correctamente asignados tanto su rol departamental como el rol **Usuario básico**. La configuración de acceso de aplicaciones ha sido revisada y documentada.

#### Verificación

Desde el panel de detalles de `usuario.rrhh`, la sección de roles debe mostrar exactamente dos roles asignados: `Rol RRHH - Contoso` y `Usuario básico`.

---

## Validación y Pruebas

Una vez completados todos los pasos, realiza las siguientes verificaciones finales para confirmar que la configuración de seguridad funciona correctamente:

### Lista de verificación de completitud

| Elemento | Verificación | Estado |
|---|---|---|
| Unidades de negocio | Existen 3 unidades secundarias bajo la raíz (Ventas, RRHH, Finanzas) | ☐ |
| Tabla personalizada | `Solicitud Interna` existe con 3 columnas en el entorno | ☐ |
| Rol Ventas | Creado con privilegios de Unidad de negocio en Cuenta y Contacto | ☐ |
| Rol RRHH | Creado con privilegios de Unidad de negocio en Contacto y Solicitud Interna | ☐ |
| Rol Finanzas | Creado con alcance Principal y secundarias en lectura | ☐ |
| Asignación usuario.ventas | Unidad: Ventas, Roles: Rol Ventas + Usuario básico | ☐ |
| Asignación usuario.rrhh | Unidad: RRHH, Roles: Rol RRHH + Usuario básico | ☐ |
| Asignación usuario.finanzas | Unidad: Finanzas, Roles: Rol Finanzas + Usuario básico | ☐ |
| Aislamiento de datos | usuario.rrhh no ve registros de Solicitud Interna de Ventas | ☐ |
| Acceso derivado del propietario | usuario.ventas ve su propio registro Solicitud-VENTAS-002 | ☐ |

### Prueba de regresión rápida

Ejecuta este comando opcional con el Power Platform CLI para listar los roles del entorno y confirmar su existencia:

```powershell
# Autenticarse en el entorno
pac auth create --url https://[nombre-org].crm.dynamics.com

# Listar los entornos disponibles
pac org list

# Seleccionar el entorno de trabajo
pac org select --environment [ID-del-entorno]
```

> Si el CLI no está disponible, la verificación visual desde el Centro de Administración es suficiente para este laboratorio.

---

## Solución de Problemas

### Problema 1: El usuario de prueba no aparece en la lista de usuarios del entorno

**Síntoma:** Al intentar agregar `usuario.ventas` (u otro usuario de prueba) al entorno desde **Configuración → Usuarios + permisos → Usuarios**, el usuario no aparece en los resultados de búsqueda o aparece con el error "No se puede agregar el usuario porque no tiene una licencia válida".

**Causa:** El usuario de prueba no tiene una licencia de Power Apps asignada en Microsoft 365, o la licencia fue asignada recientemente y la sincronización con Dataverse aún no se ha completado (puede tardar hasta 15 minutos).

**Solución:**
1. Navega a **https://admin.microsoft.com** → **Usuarios** → **Usuarios activos**.
2. Selecciona el usuario de prueba y verifica que tenga asignada una licencia de **Power Apps por usuario** o que su licencia de Microsoft 365 E5 esté activa.
3. Si la licencia acaba de asignarse, espera 10–15 minutos y vuelve a intentar agregar el usuario al entorno.
4. Si el problema persiste, intenta agregar el usuario directamente desde la URL del entorno de Dataverse: `https://[nombre-org].crm.dynamics.com/main.aspx?pagetype=entitylist&etn=systemuser` y usa el botón **Agregar usuarios**.

---

### Problema 2: Los privilegios del rol de seguridad no se guardan correctamente o los alcances se restablecen

**Síntoma:** Después de configurar los alcances de los privilegios en el editor de roles de seguridad (por ejemplo, establecer "Unidad de negocio" para Leer en la tabla Cuenta) y hacer clic en **Guardar y cerrar**, al volver a abrir el rol los alcances aparecen como "Ninguno" o con un valor diferente al configurado.

**Causa:** El editor de roles de seguridad puede tener problemas de sincronización si se realizan cambios muy rápidos entre pestañas o si la sesión del navegador tiene caché desactualizada. Adicionalmente, algunos navegadores con extensiones de bloqueo de scripts pueden interferir con la interfaz de arrastrar y hacer clic en los círculos de privilegios.

**Solución:**
1. Cierra todas las pestañas del Centro de Administración y borra la caché del navegador (Ctrl+Shift+Del → selecciona "Imágenes y archivos en caché" → Borrar datos).
2. Vuelve a abrir el Centro de Administración en una ventana nueva y navega al rol de seguridad problemático.
3. Realiza los cambios de privilegios **de forma pausada**, haciendo clic una vez en cada círculo y esperando a que el ícono cambie antes de continuar con el siguiente privilegio.
4. Usa el botón **Guardar** (no **Guardar y cerrar**) después de cada tabla configurada para ir guardando los cambios de forma incremental.
5. Si el problema persiste con Microsoft Edge, intenta usar Google Chrome o viceversa, ya que el editor de roles puede comportarse diferente según el motor de renderizado.

---

## Limpieza del Entorno

> **⚠️ Importante:** Esta práctica forma parte de una secuencia de cuatro laboratorios. **No elimines** las unidades de negocio, roles de seguridad ni usuarios configurados en esta práctica, ya que serán utilizados en las Prácticas 3 y 4. Solo realiza la limpieza si el instructor indica explícitamente que no continuarás con las prácticas siguientes.

### Limpieza parcial (solo si es necesario)

Si necesitas eliminar únicamente los registros de prueba de la tabla `Solicitud Interna` sin afectar la configuración de seguridad:

1. Navega a **https://make.powerapps.com** → entorno Sandbox → **Tablas** → **Solicitud Interna** → **Editar**.
2. Selecciona todos los registros de prueba creados durante la práctica.
3. Haz clic en **Eliminar registros** y confirma la eliminación.

### Limpieza completa (solo si no continuarás con las prácticas siguientes)

Si el instructor confirma que no se continuará con las prácticas 3 y 4, puedes revertir la configuración en el siguiente orden:

1. **Desasignar roles de usuarios:** Navega a cada usuario de prueba → **Administrar roles de seguridad** → desmarca los roles departamentales → **Guardar**.
2. **Reasignar usuarios a la unidad raíz:** Cambia la unidad de negocio de cada usuario de prueba de vuelta a la unidad raíz.
3. **Eliminar roles de seguridad personalizados:** En **Roles de seguridad**, selecciona `Rol Ventas - Contoso`, `Rol RRHH - Contoso` y `Rol Finanzas - Contoso` → **Eliminar**.
4. **Desactivar unidades de negocio:** Las unidades de negocio no pueden eliminarse directamente si tienen usuarios asociados; primero deben desactivarse desde **Unidades de negocio** → seleccionar la unidad → **Desactivar**.
5. **Eliminar la tabla personalizada:** En **https://make.powerapps.com** → **Tablas** → selecciona `Solicitud Interna` → **Eliminar tabla**.

---

## Resumen

En esta práctica implementaste un modelo completo de seguridad por unidades de negocio y roles en Dataverse para la organización ficticia Contoso Industries. Los conceptos clave aplicados fueron:

| Concepto | Lo que hiciste |
|---|---|
| **Jerarquía de unidades de negocio** | Creaste 3 unidades secundarias (Ventas, RRHH, Finanzas) bajo la unidad raíz |
| **Roles de seguridad personalizados** | Definiste privilegios diferenciados con alcances Usuario, Unidad de negocio y Principal y secundarias |
| **Tabla personalizada** | Creaste `Solicitud Interna` como superficie de datos para validar el aislamiento |
| **Asignación de usuarios** | Vinculaste usuarios de prueba a sus unidades y roles correspondientes |
| **Owner-derived Access** | Verificaste que la visibilidad de registros depende del propietario y el alcance del rol |
| **Rol Usuario básico** | Confirmaste que es el mínimo requerido para acceder al entorno |

### Puntos clave para recordar

- **Cambiar la unidad de negocio de un usuario elimina sus roles asignados.** Siempre cambia la unidad de negocio antes de asignar roles.
- **El alcance "Usuario" es el más restrictivo:** solo permite ver registros propios, independientemente de la unidad de negocio.
- **El rol Usuario básico es obligatorio** para que cualquier usuario pueda interactuar con el entorno Dataverse.
- **Los roles de seguridad son aditivos:** si un usuario tiene múltiples roles, se aplica el privilegio más permisivo entre todos ellos.
- La combinación de **unidades de negocio + alcances de roles** es el mecanismo principal para implementar el principio de mínimo privilegio en Dataverse.

### Recursos adicionales

- [Documentación oficial: Conceptos de seguridad en Microsoft Dataverse](https://learn.microsoft.com/es-es/power-platform/admin/wp-security-cds)
- [Crear y editar roles de seguridad en Dataverse](https://learn.microsoft.com/es-es/power-platform/admin/create-edit-security-role)
- [Administración de unidades de negocio en Power Platform](https://learn.microsoft.com/es-es/power-platform/admin/create-edit-business-units)
- [Referencia de privilegios de roles de seguridad](https://learn.microsoft.com/es-es/power-platform/admin/security-roles-privileges)
- [Información general sobre seguridad en Microsoft Power Platform](https://learn.microsoft.com/es-es/power-platform/admin/security/overview)

---

> **Próxima práctica — Práctica 3:** Implementación de Políticas de Prevención de Pérdida de Datos (DLP) y monitoreo de conectores en Power Platform. Asegúrate de conservar el entorno Sandbox y la configuración de seguridad creada en esta práctica.
