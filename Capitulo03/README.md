# Migración y resolución de dependencias en soluciones de Power Platform

## Metadatos

| Campo | Detalle |
|---|---|
| **Duración estimada** | 45 minutos |
| **Complejidad** | Alta |
| **Nivel Bloom** | Aplicar (*Apply*) |
| **Módulo** | 3 — Ciclo de vida de soluciones en Power Platform |
| **Versión del laboratorio** | 1.0 |

---

## Descripción General

En este laboratorio simularás el ciclo de vida completo de una solución de Power Platform en el contexto de **Contoso Industries**, migrando componentes desde un entorno de desarrollo (*Sandbox*) hacia un entorno de destino. Crearás una solución no administrada que contenga una aplicación canvas básica, un flujo de nube y una tabla personalizada en Dataverse; luego exportarás la solución en formato administrado, gestionarás las dependencias identificadas durante la importación y utilizarás el **Solution Checker** para validar la calidad de la solución antes de su despliegue en producción. Finalmente, revisarás el historial de operaciones de soluciones en el centro de administración para comprender el monitoreo operacional.

---

## Objetivos de Aprendizaje

Al finalizar este laboratorio serás capaz de:

- [ ] Crear una solución no administrada con editor personalizado, tabla de Dataverse, aplicación canvas y flujo de nube en un entorno de desarrollo.
- [ ] Exportar la solución en formato administrado y no administrado, identificando las diferencias entre ambos tipos de archivo.
- [ ] Identificar y resolver dependencias de componentes (variables de entorno y referencias a conexiones) durante la importación en un entorno destino.
- [ ] Ejecutar el Solution Checker sobre la solución y revisar e interpretar los resultados del análisis.
- [ ] Monitorear el historial de importaciones y exportaciones de soluciones desde el Power Platform Admin Center.

---

## Requisitos Previos

### Conocimientos Necesarios

- Comprensión de los conceptos de soluciones administradas y no administradas (Lección 3.1).
- Conocimiento básico de Power Apps canvas: crear una pantalla y agregar controles.
- Conocimiento básico de Power Automate: crear un flujo de nube con un desencadenador manual.
- Familiaridad con la interfaz del Power Platform Admin Center.

### Acceso y Herramientas

| Requisito | Descripción |
|---|---|
| **Tenant de Microsoft 365** | Acceso con rol de Administrador Global o Administrador de Power Platform |
| **Entorno de Desarrollo (DEV)** | Entorno Sandbox con Dataverse habilitado (creado en Práctica 1) |
| **Entorno de Destino (TEST/PROD)** | Segundo entorno con Dataverse habilitado (creado en Práctica 1 o 2) |
| **Licencias** | Power Apps y Power Automate con capacidades premium |
| **Power Platform CLI** | Versión 1.30 o superior (opcional, para tareas de línea de comandos) |
| **Visual Studio Code** | Versión 1.85 o superior con extensión Power Platform Tools (opcional) |
| **Navegador web** | Microsoft Edge 120+ o Google Chrome 120+ |


---

## Entorno de Laboratorio

### Configuración de Entornos Requeridos

| Entorno | Tipo | Propósito | URL de referencia |
|---|---|---|---|
| **Contoso-DEV** | Sandbox | Desarrollo y exportación de soluciones | `https://make.powerapps.com` |
| **Contoso-TEST** | Sandbox o Production | Importación y validación | `https://make.powerapps.com` |

### Verificación Previa del Entorno

Antes de iniciar el laboratorio, verifica que ambos entornos están operativos ejecutando los siguientes pasos:

1. Accede a [https://admin.powerplatform.microsoft.com](https://admin.powerplatform.microsoft.com).
2. En el panel lateral, selecciona **Entornos**.
3. Confirma que los entornos **Contoso-DEV** y **Contoso-TEST** (o equivalentes) aparecen con estado **Listo**.
4. Verifica que ambos tienen **Dataverse** habilitado (columna "Tipo" muestra "Sandbox" y la columna de almacenamiento muestra capacidad disponible).


## Pasos del Laboratorio

---

### Paso 1: Crear el Editor de Soluciones Personalizado

**Objetivo:** Establecer un editor (*publisher*) con prefijo de personalización propio para Contoso Industries, siguiendo la práctica recomendada de evitar el prefijo genérico `new_`.

#### Instrucciones

1. Abre tu navegador y accede a [https://make.powerapps.com](https://make.powerapps.com).

2. En la esquina superior derecha, verifica que el entorno activo es **Contoso-DEV**. Si no lo es, haz clic en el selector de entorno y selecciónalo.

3. En el panel de navegación izquierdo, haz clic en **Soluciones**.

4. En la barra de comandos superior, haz clic en **Editores** (si no ves esta opción directamente, haz clic en los tres puntos `...` o busca en el menú desplegable de la barra de comandos).

   > **Alternativa:** Puedes acceder a los editores desde **Soluciones → Nueva solución → Editor → + Editor**.

5. Haz clic en **+ Nuevo editor**.

6. Completa el formulario con los siguientes valores:

   | Campo | Valor |
   |---|---|
   | **Nombre para mostrar** | `Contoso Industries` |
   | **Nombre** | `ContosoIndustries` |
   | **Prefijo** | `contoso` |
   | **Prefijo de valor de opción** | `10000` |

7. Haz clic en **Guardar** para crear el editor.

#### Resultado Esperado

El editor **Contoso Industries** aparece en la lista de editores con el prefijo `contoso`. Todos los componentes nuevos creados dentro de las soluciones de este editor tendrán el prefijo `contoso_` en sus nombres de esquema.

#### Verificación

- Confirma que el editor aparece en la lista con el prefijo `contoso` visible.
- Verifica que el campo "Nombre" muestra `ContosoIndustries` (sin espacios).

---

### Paso 2: Crear la Solución No Administrada

**Objetivo:** Crear la solución contenedora no administrada `Contoso_GestionSolicitudes` que agrupará todos los componentes del laboratorio.

#### Instrucciones

1. En el panel de navegación izquierdo, haz clic en **Soluciones**.

2. Haz clic en **+ Nueva solución**.

3. Completa el panel lateral con los siguientes valores:

   | Campo | Valor |
   |---|---|
   | **Nombre para mostrar** | `Contoso Gestión de Solicitudes` |
   | **Nombre** | `Contoso_GestionSolicitudes` |
   | **Editor** | `Contoso Industries` (el que creaste en el Paso 1) |
   | **Versión** | `1.0.0.1` |

4. Haz clic en **Crear**.

5. La solución se crea y se abre automáticamente su vista de contenido (aparece vacía inicialmente).

#### Resultado Esperado

La solución `Contoso Gestión de Solicitudes` aparece en la lista de soluciones con estado **No administrada** y versión **1.0.0.1**. Al abrirla, muestra un contenido vacío con la barra de comandos para agregar componentes.

#### Verificación

- En la lista de soluciones, confirma que la columna **Administrada** muestra **No** para esta solución.
- Abre la solución y verifica que el editor mostrado en los detalles es **Contoso Industries**.

---

### Paso 3: Agregar una Tabla Personalizada de Dataverse

**Objetivo:** Crear una tabla personalizada `contoso_SolicitudServicio` dentro de la solución para representar el modelo de datos de Contoso Industries.

#### Instrucciones

1. Dentro de la solución `Contoso Gestión de Solicitudes`, haz clic en **+ Nuevo** en la barra de comandos.

2. En el menú desplegable, selecciona **Tabla**.

3. Selecciona **+ Nueva tabla** → **Agregar columnas y datos** (o "Establecer propiedades avanzadas" si deseas mayor control).

4. Completa la configuración de la tabla:

   | Campo | Valor |
   |---|---|
   | **Nombre para mostrar** | `Solicitud de Servicio` |
   | **Nombre para mostrar (plural)** | `Solicitudes de Servicio` |
   | **Nombre** | Se autorrellena como `contoso_SolicituddeServicio` (verifica que tenga el prefijo `contoso_`) |
   | **Tipo de registro** | Actividad: **No** |
   | **Habilitar adjuntos** | No (dejar por defecto) |

5. Haz clic en **Guardar** para crear la tabla con la columna de nombre primaria predeterminada.

6. Una vez creada la tabla, agrega una columna adicional:
   - Haz clic en **+ Nueva columna** (dentro del editor de tabla).
   - **Nombre para mostrar:** `Estado de Solicitud`
   - **Tipo de datos:** Elección (*Choice*)
   - **Sincronizar con elección global:** No (crear elección local)
   - Agrega las opciones: `Pendiente`, `En Proceso`, `Resuelta`
   - Haz clic en **Guardar**.

7. Regresa a la vista de la solución (usa el menú de navegación de migas de pan).

#### Resultado Esperado

La tabla `Solicitud de Servicio` aparece en el listado de componentes de la solución. Al expandirla, se muestran sus columnas, incluyendo `Estado de Solicitud`. El nombre de esquema de la tabla muestra el prefijo `contoso_`.

#### Verificación

- En la solución, confirma que la tabla aparece bajo la categoría **Tablas**.
- Haz clic en la tabla y verifica que la columna `contoso_estadodesolicitud` (o similar con prefijo `contoso_`) existe en el listado de columnas.

---

### Paso 4: Crear una Variable de Entorno

**Objetivo:** Agregar una variable de entorno a la solución para simular una dependencia que deberá ser resuelta durante la importación en el entorno destino.

#### Instrucciones

1. Dentro de la solución `Contoso Gestión de Solicitudes`, haz clic en **+ Nuevo**.

2. Selecciona **Más** → **Variable de entorno**.

3. Completa el formulario:

   | Campo | Valor |
   |---|---|
   | **Nombre para mostrar** | `URL del Portal de Solicitudes` |
   | **Nombre** | `contoso_URLPortalSolicitudes` |
   | **Tipo de datos** | Texto |
   | **Valor predeterminado** | `https://dev.contoso.com/solicitudes` |
   | **Valor actual** | *(dejar vacío por ahora)* |

4. Haz clic en **Guardar**.

#### Resultado Esperado

La variable de entorno `URL del Portal de Solicitudes` aparece en la solución bajo la categoría **Variables de entorno**. Esta variable será una dependencia que el administrador del entorno destino deberá configurar con el valor correcto para ese entorno.

#### Verificación

- Confirma que la variable aparece en la solución con tipo **Texto**.
- Verifica que el valor predeterminado es `https://dev.contoso.com/solicitudes`.

---

### Paso 5: Crear la Aplicación Canvas

**Objetivo:** Agregar una aplicación canvas básica a la solución que utilice la tabla personalizada creada en el Paso 3.

#### Instrucciones

1. Dentro de la solución, haz clic en **+ Nuevo** → **Aplicación** → **Aplicación de lienzo**.

2. En el diálogo de creación, selecciona:
   - **Nombre de la aplicación:** `Gestión de Solicitudes - Contoso`
   - **Formato:** Tableta

3. Haz clic en **Crear**. Se abrirá Power Apps Studio en una nueva pestaña del navegador.

4. En Power Apps Studio, agrega un origen de datos:
   - En el panel izquierdo, haz clic en el icono de **Datos** (cilindro de base de datos).
   - Haz clic en **+ Agregar datos**.
   - Busca `Solicitud` y selecciona la tabla **Solicitudes de Servicio** de Dataverse.

5. Agrega una pantalla básica:
   - En **Screen1**, inserta un control **Galería vertical** desde el menú **Insertar → Galería → Vertical**.
   - Configura el origen de datos de la galería: en la propiedad **Items**, escribe `'Solicitudes de Servicio'`.
   - Agrega una etiqueta de título en la pantalla: **Insertar → Etiqueta de texto**, con el texto `"Solicitudes de Servicio - Contoso Industries"`.

6. Guarda la aplicación:
   - Presiona `Ctrl + S` o haz clic en **Archivo → Guardar**.
   - Selecciona **Guardar**.

7. Cierra la pestaña de Power Apps Studio y regresa a la pestaña de la solución.

8. Actualiza la vista de la solución (botón **Actualizar** en la barra de comandos o `F5`). La aplicación debería aparecer en la lista de componentes.

#### Resultado Esperado

La aplicación `Gestión de Solicitudes - Contoso` aparece en la solución bajo la categoría **Aplicaciones**. La aplicación tiene una pantalla con una galería conectada a la tabla `Solicitudes de Servicio`.

#### Verificación

- Confirma que la aplicación aparece en la solución.
- Haz clic en los tres puntos `...` junto a la aplicación y selecciona **Detalles**; verifica que el editor es **Contoso Industries**.

---

### Paso 6: Crear el Flujo de Nube

**Objetivo:** Agregar un flujo de Power Automate a la solución que utilice la tabla de Dataverse y demuestre una dependencia de conexión durante la importación.

#### Instrucciones

1. Dentro de la solución, haz clic en **+ Nuevo** → **Automatización** → **Flujo de nube** → **Automatizado**.

2. En el diálogo de configuración del flujo:
   - **Nombre del flujo:** `Notificar Nueva Solicitud - Contoso`
   - **Desencadenador:** Busca `Dataverse` y selecciona **Cuando se agrega, modifica o elimina una fila** (When a row is added, modified or deleted).

3. Haz clic en **Crear**.

4. En el editor de flujos, configura el desencadenador:
   - **Tipo de cambio:** Agregado (*Added*)
   - **Nombre de tabla:** `Solicitudes de Servicio`
   - **Ámbito:** Organización

5. Agrega un paso de acción:
   - Haz clic en **+ Nuevo paso**.
   - Busca `Enviar un correo electrónico` y selecciona la acción de **Office 365 Outlook → Enviar un correo electrónico (V2)**.
   - Configura los campos básicos:
     - **Para:** `admin@[tu-tenant].onmicrosoft.com` (usa el correo del administrador)
     - **Asunto:** `Nueva Solicitud de Servicio Registrada`
     - **Cuerpo:** `Se ha registrado una nueva solicitud de servicio en el sistema Contoso.`

6. Guarda el flujo haciendo clic en **Guardar** en la esquina superior derecha.

7. Cierra la pestaña del editor de flujos y regresa a la solución. Actualiza la vista.

#### Resultado Esperado

El flujo `Notificar Nueva Solicitud - Contoso` aparece en la solución bajo la categoría **Flujos de nube**. Este flujo tiene una dependencia de conexión a **Office 365 Outlook** que deberá ser reconectada al importar en el entorno destino.

#### Verificación

- Confirma que el flujo aparece en la solución con estado **Activado** o **Desactivado**.
- Verifica que la solución ahora contiene los cuatro tipos de componentes: Tabla, Variable de entorno, Aplicación y Flujo.

---

### Paso 7: Exportar la Solución (No Administrada y Administrada)

**Objetivo:** Exportar la solución en ambos formatos para comprender las diferencias y obtener el archivo administrado que se utilizará en la importación.

#### Instrucciones

**Parte A: Exportar como No Administrada**

1. Regresa a la lista principal de **Soluciones** (clic en "Soluciones" en el panel de navegación).

2. Selecciona la solución `Contoso Gestión de Solicitudes` (marca la casilla a la izquierda del nombre).

3. En la barra de comandos, haz clic en **Exportar solución**.

4. En el panel lateral de exportación:
   - Revisa el resumen de componentes que se exportarán.
   - Haz clic en **Siguiente**.

5. En la pantalla de opciones de exportación:
   - **Versión:** `1.0.0.1` (confirma que es correcta)
   - **Exportar como:** Selecciona **No administrada**
   - Haz clic en **Exportar**.

6. El archivo `.zip` se descargará automáticamente. El nombre será similar a:
   `Contoso_GestionSolicitudes_1_0_0_1.zip`

7. Guarda este archivo en una carpeta identificada, por ejemplo: `C:\LabFiles\Soluciones\NoAdministrada\`

**Parte B: Exportar como Administrada**

8. Repite los pasos 2 al 5, pero esta vez selecciona **Administrada** en el paso de opciones de exportación.

9. El archivo descargado tendrá el nombre:
   `Contoso_GestionSolicitudes_1_0_0_1_managed.zip`

10. Guarda este archivo en: `C:\LabFiles\Soluciones\Administrada\`

**Parte C: Comparar los archivos (Opcional pero recomendado)**

11. Abre el Explorador de archivos y navega a ambas carpetas.

12. Observa los tamaños de archivo: el archivo administrado suele ser ligeramente diferente al no administrado.

13. Para inspeccionar el contenido, copia cada archivo `.zip` y renómbralo con extensión `.zip` si es necesario, luego extrae su contenido:
    - Abre una terminal (PowerShell o CMD) y ejecuta:

```powershell
# Extraer el archivo no administrado para inspección
Expand-Archive -Path "C:\LabFiles\Soluciones\NoAdministrada\Contoso_GestionSolicitudes_1_0_0_1.zip" -DestinationPath "C:\LabFiles\Soluciones\Inspeccion\NoAdmin" -Force

# Extraer el archivo administrado para inspección
Expand-Archive -Path "C:\LabFiles\Soluciones\Administrada\Contoso_GestionSolicitudes_1_0_0_1_managed.zip" -DestinationPath "C:\LabFiles\Soluciones\Inspeccion\Admin" -Force
```

14. Abre la carpeta extraída y localiza el archivo `solution.xml`. Ábrelo con un editor de texto (Notepad o VS Code) y busca el atributo `Managed`:

```xml
<!-- En la solución NO administrada verás: -->
<Managed>0</Managed>

<!-- En la solución ADMINISTRADA verás: -->
<Managed>1</Managed>
```

#### Resultado Esperado

Tienes dos archivos `.zip` en tu equipo:
- `Contoso_GestionSolicitudes_1_0_0_1.zip` — Solución no administrada (para uso en DEV).
- `Contoso_GestionSolicitudes_1_0_0_1_managed.zip` — Solución administrada (para despliegue en TEST/PROD).

La diferencia en el atributo `Managed` del archivo `solution.xml` confirma el tipo de solución.

#### Verificación

- Confirma que ambos archivos existen en las rutas especificadas.
- Verifica que el atributo `<Managed>` es `0` en el no administrado y `1` en el administrado.

---

### Paso 8: Ejecutar el Solution Checker

**Objetivo:** Analizar la calidad de la solución con el Solution Checker integrado en Power Apps para identificar problemas antes de la migración a producción.

#### Instrucciones

1. En [https://make.powerapps.com](https://make.powerapps.com), asegúrate de estar en el entorno **Contoso-DEV**.

2. Navega a **Soluciones** en el panel izquierdo.

3. Localiza la solución `Contoso Gestión de Solicitudes`.

4. Haz clic en los tres puntos `...` a la derecha del nombre de la solución.

5. En el menú contextual, selecciona **Comprobador de soluciones** → **Ejecutar**.

6. Aparecerá una notificación indicando que el análisis está en curso. El proceso puede tardar entre 2 y 5 minutos dependiendo del tamaño de la solución.

   > **Nota:** Puedes continuar trabajando mientras el análisis se ejecuta en segundo plano. Recibirás una notificación cuando finalice.

7. Una vez completado el análisis, haz clic en los tres puntos `...` nuevamente y selecciona **Comprobador de soluciones** → **Ver resultados**.

8. Revisa los resultados del análisis. Presta atención a:
   - **Errores (rojo):** Problemas críticos que deben corregirse antes de la migración.
   - **Advertencias (amarillo):** Problemas que deberían corregirse para seguir las mejores prácticas.
   - **Informativo (azul):** Sugerencias de optimización.

9. Para cada resultado, observa:
   - **Nombre del componente afectado**
   - **Tipo de problema**
   - **Descripción del problema**
   - **Nivel de gravedad**
   - **Enlace a documentación de corrección** (si está disponible)

10. Documenta en una tabla los resultados encontrados (ejemplo de formato):

    | Componente | Tipo de Problema | Gravedad | Acción Recomendada |
    |---|---|---|---|
    | `Notificar Nueva Solicitud` | Uso de conexiones hardcoded | Advertencia | Usar variables de entorno |
    | `Gestión de Solicitudes` | *(resultado real del análisis)* | *(nivel)* | *(acción)* |

#### Resultado Esperado

El Solution Checker completa el análisis y muestra un informe con los resultados. Es posible que encuentres advertencias relacionadas con el flujo (por ejemplo, uso de valores codificados en lugar de variables de entorno) o con la aplicación canvas. El análisis confirma que la solución puede ser exportada, aunque con recomendaciones de mejora.

#### Verificación

- Confirma que el análisis se completó (el estado muestra "Resultados disponibles" o similar).
- Verifica que puedes ver el informe detallado con al menos una categoría de resultados.
- Anota si hay errores críticos que impedirían la importación.

---

### Paso 9: Importar la Solución Administrada en el Entorno Destino

**Objetivo:** Importar el archivo administrado en el entorno Contoso-TEST y gestionar las dependencias identificadas durante el proceso de importación.

#### Instrucciones

1. En [https://make.powerapps.com](https://make.powerapps.com), cambia al entorno **Contoso-TEST** usando el selector de entorno en la esquina superior derecha.

2. En el panel de navegación izquierdo, haz clic en **Soluciones**.

3. En la barra de comandos, haz clic en **Importar solución**.

4. En el panel lateral de importación, haz clic en **Examinar** y selecciona el archivo:
   `Contoso_GestionSolicitudes_1_0_0_1_managed.zip`

5. Haz clic en **Siguiente**.

6. Power Platform analizará el paquete y mostrará la información de la solución. Revisa:
   - Nombre: `Contoso Gestión de Solicitudes`
   - Versión: `1.0.0.1`
   - Editor: `Contoso Industries`

7. Haz clic en **Siguiente** nuevamente.

**Gestión de Dependencias — Variables de Entorno:**

8. En la pantalla de **Variables de entorno**, verás la variable `URL del Portal de Solicitudes` listada con el valor predeterminado `https://dev.contoso.com/solicitudes`.

9. En el campo **Valor actual**, ingresa el valor correcto para el entorno TEST:
   `https://test.contoso.com/solicitudes`

   > **Explicación:** Esta es la gestión de dependencias en acción. El valor predeterminado era el del entorno de desarrollo; en el entorno de destino debes proporcionar el valor correcto para ese contexto específico.

**Gestión de Dependencias — Conexiones:**

10. En la pantalla de **Conexiones**, verás la conexión de **Office 365 Outlook** listada como requerida por el flujo.

11. Si ya existe una conexión disponible en el entorno TEST, selecciónala del desplegable.

12. Si no existe, haz clic en **Crear nueva conexión**:
    - Se abrirá una nueva pestaña para crear la conexión.
    - Inicia sesión con las credenciales del usuario propietario del flujo en el entorno TEST.
    - Regresa a la pestaña de importación y actualiza la lista de conexiones.
    - Selecciona la conexión recién creada.

13. Haz clic en **Importar**.

14. El proceso de importación comenzará. Espera a que se complete (puede tardar 1-3 minutos). Verás una barra de progreso y mensajes de estado.

#### Resultado Esperado

La solución `Contoso Gestión de Solicitudes` aparece en la lista de soluciones del entorno **Contoso-TEST** con:
- Estado: **Administrada** (la columna "Administrada" muestra "Sí").
- Versión: `1.0.0.1`
- Todos los componentes importados correctamente: tabla, variable de entorno, aplicación y flujo.

Si la importación falla por alguna dependencia no resuelta, se mostrará un mensaje de error con detalles sobre el componente problemático.

#### Verificación

- En el entorno TEST, confirma que la solución aparece con el indicador **Administrada = Sí**.
- Abre la solución y verifica que los cuatro componentes están presentes.
- Navega a **Variables de entorno** y confirma que el valor actual es `https://test.contoso.com/solicitudes`.
- Verifica que el flujo tiene el estado **Activado** (puede requerir activación manual tras la importación).

---

### Paso 10: Monitorear el Historial de Operaciones de Soluciones

**Objetivo:** Revisar el registro de operaciones de importación y exportación de soluciones en el Power Platform Admin Center para comprender las capacidades de monitoreo operacional.

#### Instrucciones

1. Abre una nueva pestaña del navegador y accede al **Power Platform Admin Center**:
   [https://admin.powerplatform.microsoft.com](https://admin.powerplatform.microsoft.com)

2. En el panel de navegación izquierdo, haz clic en **Entornos**.

3. Haz clic en el nombre del entorno **Contoso-TEST** para abrir sus detalles.

4. En la página de detalles del entorno, busca la sección **Historial** o navega a través de las opciones disponibles. Dependiendo de la versión del Admin Center, puedes encontrar el historial de soluciones de las siguientes formas:

   **Opción A — Desde el Admin Center:**
   - En la página de detalles del entorno, haz clic en **Recursos** → **Dynamics 365 apps** o busca la opción **Historial de soluciones**.

   **Opción B — Desde Power Apps:**
   - Regresa a [https://make.powerapps.com](https://make.powerapps.com) en el entorno **Contoso-TEST**.
   - Ve a **Soluciones** en el panel izquierdo.
   - Haz clic en **Historial de soluciones** en la barra de comandos superior (puede estar bajo el menú `...` o como opción directa).

5. En la vista de **Historial de soluciones**, revisa las entradas registradas. Para cada operación, observa:

   | Campo | Descripción |
   |---|---|
   | **Nombre de la solución** | Nombre de la solución operada |
   | **Operación** | Tipo: Importar, Exportar, Publicar, Eliminar |
   | **Estado** | Completado, Error, En progreso |
   | **Inicio** | Fecha y hora de inicio de la operación |
   | **Fin** | Fecha y hora de finalización |
   | **Versión** | Versión de la solución |

6. Haz clic en la entrada de la importación de `Contoso Gestión de Solicitudes` para ver los detalles completos de la operación.

7. En los detalles, revisa:
   - Los componentes que fueron procesados durante la importación.
   - Si hubo advertencias o errores no críticos durante el proceso.
   - El tiempo total de la operación.

8. Opcionalmente, verifica también el historial en el entorno **Contoso-DEV** para ver las exportaciones realizadas.

#### Resultado Esperado

El historial de soluciones muestra al menos dos entradas para el entorno TEST:
- Una operación de tipo **Importar** con estado **Completado** para `Contoso Gestión de Solicitudes`.
- Detalles de los componentes procesados durante la importación.

#### Verificación

- Confirma que la operación de importación aparece con estado **Completado**.
- Verifica que el timestamp de la operación corresponde al momento en que realizaste la importación.
- Documenta el tiempo total que tomó la operación de importación (dato útil para planificar ventanas de mantenimiento).

---

## Validación y Pruebas

Una vez completados todos los pasos, realiza las siguientes verificaciones para confirmar que el laboratorio fue exitoso:

### Lista de Verificación Final

| # | Verificación | Método | Resultado Esperado |
|---|---|---|---|
| 1 | Editor `Contoso Industries` creado con prefijo `contoso` | Power Apps → Soluciones → Editores | Editor visible con prefijo correcto |
| 2 | Solución no administrada creada con 4 componentes | Power Apps → Soluciones → Contoso-DEV | Tabla, App, Flujo y Variable de entorno presentes |
| 3 | Archivo no administrado exportado | Explorador de archivos | `.zip` sin `_managed` en el nombre |
| 4 | Archivo administrado exportado | Explorador de archivos | `.zip` con `_managed` en el nombre |
| 5 | `solution.xml` confirma tipo de solución | Editor de texto sobre archivo extraído | `<Managed>0</Managed>` y `<Managed>1</Managed>` respectivamente |
| 6 | Solution Checker ejecutado y resultados revisados | Power Apps → Soluciones → `...` → Comprobador | Informe disponible con resultados |
| 7 | Solución importada en entorno TEST como administrada | Power Apps → Contoso-TEST → Soluciones | Solución con indicador "Administrada = Sí" |
| 8 | Variable de entorno con valor correcto para TEST | Power Apps → Contoso-TEST → Solución → Variables | Valor `https://test.contoso.com/solicitudes` |
| 9 | Historial de operaciones revisado | Admin Center o Power Apps | Entrada de importación con estado "Completado" |

### Prueba Funcional de la Solución Importada

1. En el entorno **Contoso-TEST**, navega a **Soluciones** → `Contoso Gestión de Solicitudes`.
2. Haz clic en la aplicación `Gestión de Solicitudes - Contoso`.
3. Selecciona **Reproducir** para ejecutar la aplicación.
4. Verifica que la pantalla se carga correctamente con la galería de solicitudes (puede estar vacía si no hay datos en TEST, lo cual es normal).
5. Intenta activar el flujo `Notificar Nueva Solicitud - Contoso` desde la solución (clic en `...` → **Activar**) y confirma que se activa sin errores de conexión.

---

## Recursos Adicionales

| Recurso | URL |
|---|---|
| Documentación oficial: Soluciones en Power Platform | [https://learn.microsoft.com/es-es/power-platform/alm/solution-concepts-alm](https://learn.microsoft.com/es-es/power-platform/alm/solution-concepts-alm) |
| Exportar e importar soluciones | [https://learn.microsoft.com/es-es/power-apps/maker/data-platform/import-update-export-solutions](https://learn.microsoft.com/es-es/power-apps/maker/data-platform/import-update-export-solutions) |
| Variables de entorno en soluciones | [https://learn.microsoft.com/es-es/power-apps/maker/data-platform/environmentvariables](https://learn.microsoft.com/es-es/power-apps/maker/data-platform/environmentvariables) |
| Solution Checker — Referencia de reglas | [https://learn.microsoft.com/es-es/power-apps/maker/data-platform/use-powerapps-checker](https://learn.microsoft.com/es-es/power-apps/maker/data-platform/use-powerapps-checker) |
| Power Platform CLI — Comandos de solución | [https://learn.microsoft.com/es-es/power-platform/developer/cli/reference/solution](https://learn.microsoft.com/es-es/power-platform/developer/cli/reference/solution) |
| Capas de solución en Dataverse | [https://learn.microsoft.com/es-es/power-apps/maker/data-platform/solution-layers](https://learn.microsoft.com/es-es/power-apps/maker/data-platform/solution-layers) |
| Mejores prácticas de ALM | [https://learn.microsoft.com/es-es/power-platform/alm/best-practices-alm](https://learn.microsoft.com/es-es/power-platform/alm/best-practices-alm) |

---
*Lab 03-00-01 — Contoso Industries · Power Platform ALM · Versión 1.0*
