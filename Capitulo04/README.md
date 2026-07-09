# Gobierno integral de Power Platform con monitoreo, seguridad y CoE Starter Kit

## Metadatos

| Campo            | Valor                                                                 |
|------------------|-----------------------------------------------------------------------|
| **Duración**     | 35 minutos                                                            |
| **Complejidad**  | Alta                                                                  |
| **Nivel Bloom**  | Aplicar (Apply)                                                       |
| **Módulo**       | Capítulo 4 — Flujos y gobierno en Power Platform                     |
| **Versión**      | 1.0                                                                   |

---

## Descripción general

En esta práctica integradora, el estudiante implementará el marco de gobierno más completo del curso para el tenant ficticio de **Contoso Industries**, utilizando el CoE Starter Kit de Microsoft Power Platform. Se explorarán los componentes principales del kit (Core, Governance, Nurture e Innovation Backlog), se configurarán los flujos de sincronización de inventario y se analizará el panel de Power BI para obtener visibilidad sobre aplicaciones, flujos y conectores del tenant. La práctica conecta directamente con los conceptos de la Lección 4.1, donde se aprendió que los Cloud Flows (Automated, Instant y Scheduled), los Business Process Flows y los Desktop Flows son los tres pilares de automatización de Power Platform, todos ellos sujetos a gobierno y monitoreo. Al finalizar, el estudiante elaborará un plan de gobierno integral que consolide todos los aprendizajes del curso.

---

## Objetivos de aprendizaje

Al completar este laboratorio, el estudiante será capaz de:

- [ ] Explorar la arquitectura y los componentes principales del CoE Starter Kit, identificando las aplicaciones, flujos y paneles incluidos en el kit.
- [ ] Instalar y configurar el componente **Core** del CoE Starter Kit en un entorno dedicado de gobierno, habilitando la sincronización de inventario de recursos de Power Platform.
- [ ] Utilizar el panel de Power BI del CoE Starter Kit para analizar el inventario de aplicaciones, flujos y conectores utilizados en el tenant.
- [ ] Configurar y revisar los flujos de nube de gobierno del CoE Starter Kit para automatizar la comunicación con creadores y la gestión del ciclo de vida de recursos.
- [ ] Diseñar un plan de gobierno integral para Power Platform que integre políticas DLP, monitoreo con CoE, roles de administración y procesos de aprobación de nuevas soluciones.

---

## Requisitos previos

### Conocimientos requeridos

| Área                              | Nivel esperado                                                                                 |
|-----------------------------------|-----------------------------------------------------------------------------------------------|
| Power Platform Admin Center       | Navegación básica; haber completado Prácticas 1, 2 y 3 del curso                             |
| Power Automate Cloud Flows        | Comprensión de los tipos de flujos (Automated, Instant, Scheduled) — Lección 4.1             |
| Microsoft Dataverse               | Creación de entornos con Dataverse habilitado                                                 |
| Power Apps (Model-driven Apps)    | Apertura y navegación básica de aplicaciones basadas en modelos                              |
| Power BI                          | Apertura de archivos `.pbix` en Power BI Desktop o conexión a Power BI Service               |
| Políticas DLP                     | Comprensión de conectores bloqueados/permitidos — Prácticas anteriores                       |

### Acceso y licencias requeridos

| Recurso                                           | Requerimiento                                                                                     |
|---------------------------------------------------|---------------------------------------------------------------------------------------------------|
| Tenant de Microsoft 365                           | Administrador Global **o** Administrador de Power Platform                                        |
| Entorno CoE dedicado                              | Dataverse habilitado, creado en Práctica 1 o provisto por el instructor                          |
| Licencias Power Automate                          | Premium (acceso a conectores premium y Dataverse)                                                 |
| CoE Starter Kit descargado                        | Desde [https://aka.ms/CoEStarterKitDownload](https://aka.ms/CoEStarterKitDownload)               |
| Power BI Desktop **o** Power BI Service           | Versión actual para visualización del panel del CoE                                               |
| Usuarios de prueba (mínimo 3)                     | Con licencias de Power Platform asignadas — provistos por el instructor                           |

> **⚠️ Nota sobre sincronización:** La primera sincronización del CoE Starter Kit puede tomar entre 30 y 60 minutos. El instructor debe haber ejecutado la sincronización inicial antes de la sesión. Los estudiantes comenzarán desde la etapa de configuración y exploración del inventario ya sincronizado.

---

## Entorno de laboratorio

### Hardware recomendado

| Componente       | Mínimo                                    | Recomendado                              |
|------------------|-------------------------------------------|------------------------------------------|
| Procesador       | Intel Core i5 8ª gen / AMD Ryzen 5       | Intel Core i7 10ª gen o superior         |
| Memoria RAM      | 8 GB                                      | 16 GB                                    |
| Almacenamiento   | 10 GB libres                              | 20 GB libres                             |
| Resolución       | 1366 × 768                                | 1920 × 1080                              |
| Conexión         | 10 Mbps                                   | 25 Mbps o superior                       |

### Software requerido

| Software                          | Versión mínima        | Observación                                              |
|-----------------------------------|-----------------------|----------------------------------------------------------|
| Microsoft Edge / Google Chrome    | 120.0 o superior      | Para acceso a portales de Power Platform                 |
| Power Platform Admin Center       | SaaS                  | [https://aka.ms/ppac](https://aka.ms/ppac)               |
| Power Apps                        | SaaS                  | [https://make.powerapps.com](https://make.powerapps.com) |
| Power Automate                    | SaaS                  | [https://make.powerautomate.com](https://make.powerautomate.com) |
| Power Platform CLI (pac CLI)      | 1.30 o superior       | Para importación de soluciones por línea de comandos     |
| Visual Studio Code                | 1.85 o superior       | Editor opcional para revisar archivos de solución        |
| Power BI Desktop                  | Versión actual        | Para apertura del panel `.pbix` del CoE                  |
| Microsoft .NET SDK                | 6.0 o superior        | Requerido por pac CLI                                    |

### Configuración inicial del entorno

Antes de iniciar los pasos del laboratorio, verifique que el entorno dedicado de gobierno esté disponible y que el CoE Starter Kit esté descargado localmente.

```powershell
# Verificar versión instalada de pac CLI
pac --version

# Autenticarse en el tenant de Microsoft 365
pac auth create --url https://[su-entorno-coe].crm.dynamics.com

# Verificar la autenticación activa
pac auth list

# Listar los entornos disponibles en el tenant
pac env list
```

> **Resultado esperado de `pac env list`:** Debe aparecer el entorno de gobierno (p. ej., `CoE-Contoso`) junto con los entornos de Desarrollo y Producción creados en prácticas anteriores.

---

## Pasos del laboratorio

---

### Paso 1 — Explorar la arquitectura del CoE Starter Kit

**Objetivo:** Comprender los cuatro componentes principales del CoE Starter Kit y su relación con los tipos de flujos de Power Platform aprendidos en la Lección 4.1.

#### Instrucciones

1. Abra el navegador y diríjase a la documentación oficial del CoE Starter Kit:
   ```
   https://learn.microsoft.com/es-es/power-platform/guidance/coe/starter-kit
   ```

2. En la documentación, localice la sección **"Componentes del CoE Starter Kit"** e identifique los cuatro módulos principales. Tome nota de los siguientes datos en la tabla de su cuaderno o documento de trabajo:

   | Componente           | Propósito principal                                      | Tipo de flujo predominante (Lección 4.1) |
   |----------------------|----------------------------------------------------------|------------------------------------------|
   | **Core**             | Inventario y sincronización de recursos del tenant       | Cloud Flows — Scheduled / Automated      |
   | **Governance**       | Aplicación de políticas y notificaciones a creadores     | Cloud Flows — Automated                  |
   | **Nurture**          | Capacitación y adopción de creadores                     | Cloud Flows — Instant / Automated        |
   | **Innovation Backlog** | Gestión de ideas y solicitudes de nuevas soluciones   | Business Process Flow + Cloud Flows      |

3. En el portal de Power Apps ([https://make.powerapps.com](https://make.powerapps.com)), seleccione el entorno **CoE-Contoso** (o el entorno de gobierno provisto por el instructor) en el selector de entorno (esquina superior derecha).

4. Navegue a **Soluciones** en el panel izquierdo. Verifique que las siguientes soluciones estén presentes (instaladas previamente por el instructor):
   - `Center of Excellence - Core Components`
   - `Center of Excellence - Governance Components`
   - `Center of Excellence - Nurture Components`

5. Haga clic en **Center of Excellence - Core Components** para expandir su contenido. Identifique y anote cuántos elementos de cada tipo contiene la solución:

   ```
   Tablas (Entities/Tables): ____
   Flujos de nube (Cloud Flows): ____
   Aplicaciones de lienzo (Canvas Apps): ____
   Aplicaciones basadas en modelos (Model-driven Apps): ____
   Paneles (Dashboards): ____
   ```

**Resultado esperado:** El estudiante debe identificar que el componente Core contiene aproximadamente 15-20 flujos de nube de tipo **Scheduled** y **Automated** (según la Lección 4.1), cuyo propósito es sincronizar automáticamente el inventario de recursos del tenant en las tablas de Dataverse del entorno CoE.

**Verificación:**
- ✅ El entorno `CoE-Contoso` aparece en el selector de entornos de Power Apps.
- ✅ Las tres soluciones CoE están visibles en la sección **Soluciones**.
- ✅ La solución Core contiene flujos de nube en estado **Activado** (verde).

---

### Paso 2 — Configurar las variables de entorno del CoE Starter Kit

**Objetivo:** Configurar las variables de entorno requeridas por el componente Core para que los flujos de sincronización puedan conectarse correctamente al tenant.

#### Instrucciones

1. Dentro de la solución **Center of Excellence - Core Components**, haga clic en el filtro **Tipo** y seleccione **Variable de entorno** para ver únicamente las variables de entorno disponibles.

2. Localice y configure las siguientes variables de entorno críticas. Para cada una, haga clic en los **tres puntos (...)** → **Editar** → ingrese el valor correspondiente:

   | Variable de entorno                          | Valor a configurar                                                        |
   |----------------------------------------------|---------------------------------------------------------------------------|
   | `Admin eMail`                                | Correo electrónico del administrador del tenant (su cuenta de admin)      |
   | `Also Delete from CoE`                       | `Yes`                                                                     |
   | `Approval Admin`                             | Correo electrónico del administrador para aprobaciones                    |
   | `CoE System User`                            | UPN del usuario de servicio del CoE (provisto por el instructor)          |
   | `Power Platform Maker Microsoft 365 Group`   | ID del grupo de Microsoft 365 para creadores (provisto por el instructor) |

3. Para obtener el **ID del entorno CoE** (necesario para algunas variables), ejecute el siguiente comando en la terminal con pac CLI:

   ```powershell
   # Obtener el ID del entorno de gobierno CoE
   pac env list --filter "CoE"
   ```

   Copie el valor del campo `Environment ID` del resultado.

4. Regrese al portal de Power Apps y localice la variable `Environment ID` (si existe en la versión instalada). Ingrese el ID copiado en el paso anterior.

5. Una vez configuradas las variables, navegue a **Flujos de nube** dentro de la solución Core. Verifique que los siguientes flujos clave estén en estado **Activado**:

   ```
   ✅ CORE - Sync Flows v3 (Scheduled)
   ✅ CORE - Admin | Sync Template v3 (Automated)
   ✅ CORE - Get M365 Service Health (Scheduled)
   ```

   > **Nota:** Si algún flujo aparece en estado **Desactivado**, haga clic en los tres puntos (**...**) → **Activar**.

6. Para verificar que las conexiones de los flujos están correctamente configuradas, haga clic en el flujo **CORE - Sync Flows v3** y revise la sección **Conexiones**. Debe mostrar conexiones activas (sin errores en rojo) para:
   - **Dataverse** — conectado con la cuenta de administrador
   - **Office 365 Users** — conectado con la cuenta de administrador
   - **Power Platform for Admins** — conectado con la cuenta de administrador

**Resultado esperado:** Todas las variables de entorno críticas tienen valores configurados y los flujos de sincronización del Core están en estado **Activado** con conexiones activas.

**Verificación:**
- ✅ La variable `Admin eMail` muestra el correo del administrador sin errores de validación.
- ✅ Los flujos Scheduled del Core están activados y muestran una próxima ejecución programada.
- ✅ No hay conexiones en estado de error (ícono rojo) en los flujos del Core.

---

### Paso 3 — Ejecutar la sincronización de inventario y verificar los datos

**Objetivo:** Ejecutar manualmente el flujo de sincronización del CoE Starter Kit y verificar que los datos del inventario del tenant se almacenan correctamente en las tablas de Dataverse.

#### Instrucciones

1. Desde el portal de Power Automate ([https://make.powerautomate.com](https://make.powerautomate.com)), asegúrese de estar en el entorno **CoE-Contoso**.

2. Navegue a **Mis flujos** → **Flujos de solución** y localice el flujo **CORE - Sync Flows v3**.

3. Haga clic en el flujo para abrir su detalle y luego haga clic en el botón **Ejecutar** (ícono de triángulo/play) para iniciar una ejecución manual. Confirme la ejecución en el diálogo que aparece.

   > **⚠️ Advertencia:** Si el instructor ya ejecutó la sincronización inicial, omita este paso para evitar duplicación de datos. Proceda directamente al punto 4.

4. Mientras el flujo se ejecuta (o después de que el instructor haya confirmado la sincronización inicial), navegue a la aplicación de inventario del CoE. En el portal de Power Apps, vaya a **Aplicaciones** y busque la aplicación **CoE Admin - Power Platform Admin View** (Model-driven App).

5. Abra la aplicación **Power Platform Admin View**. En el panel de navegación izquierdo, explore las siguientes secciones y anote la cantidad de registros encontrados en cada tabla:

   | Tabla del CoE                  | Cantidad de registros | Observación                             |
   |--------------------------------|-----------------------|-----------------------------------------|
   | **Environments** (Entornos)    | ____                  | Todos los entornos del tenant           |
   | **Apps** (Aplicaciones)        | ____                  | Canvas Apps y Model-driven Apps         |
   | **Flows** (Flujos)             | ____                  | Cloud Flows de todos los entornos       |
   | **Connectors** (Conectores)    | ____                  | Conectores utilizados en el tenant      |
   | **Makers** (Creadores)         | ____                  | Usuarios que han creado recursos        |

6. En la sección **Flows**, aplique un filtro por **Flow Type** para identificar cuántos flujos de cada tipo existen en el tenant (conectando con la taxonomía de la Lección 4.1):

   ```
   Automated Flows: ____
   Instant Flows:   ____
   Scheduled Flows: ____
   Business Process Flows: ____
   Desktop Flows:   ____
   ```

7. Identifique al menos **2 aplicaciones huérfanas** (aplicaciones cuyo creador ya no está activo en el tenant o cuyo campo `Owner` aparece como `N/A` o `Unknown`). Anote sus nombres para utilizarlos en el Paso 5.

**Resultado esperado:** La tabla **Environments** muestra todos los entornos del tenant (incluyendo los creados en las Prácticas 1, 2 y 3). La tabla **Flows** muestra una distribución de flujos por tipo que refleja los recursos existentes en el tenant.

**Verificación:**
- ✅ La aplicación **Power Platform Admin View** abre sin errores.
- ✅ La tabla **Environments** contiene al menos 3 registros (entornos de Desarrollo, Producción y CoE).
- ✅ La tabla **Flows** contiene registros con el campo `Flow Type` correctamente clasificado.
- ✅ Se han identificado al menos 2 aplicaciones candidatas para revisión de gobierno.

---

### Paso 4 — Analizar el inventario con el panel de Power BI del CoE

**Objetivo:** Utilizar el panel de Power BI del CoE Starter Kit para analizar visualmente el inventario de recursos, identificar tendencias de uso y detectar riesgos de gobierno.

#### Instrucciones

1. Localice el archivo de panel de Power BI del CoE Starter Kit en la carpeta donde descargó el kit. El archivo se llama:
   ```
   CoEDashboard.pbix
   ```
   Generalmente se encuentra en la subcarpeta `PowerBI` del paquete descargado.

2. Abra el archivo `CoEDashboard.pbix` con **Power BI Desktop**. Al abrirlo por primera vez, aparecerá un diálogo solicitando la URL del entorno de Dataverse.

3. En el diálogo de parámetros, ingrese la URL del entorno CoE en el siguiente formato:
   ```
   https://[nombre-entorno-coe].crm.dynamics.com
   ```
   Haga clic en **Cargar** y autentíquese con las credenciales del administrador cuando se solicite.

4. Una vez cargados los datos, explore las siguientes páginas del panel y complete la tabla de análisis:

   **Página: Overview (Resumen General)**
   - Total de aplicaciones en el tenant: ____
   - Total de flujos activos: ____
   - Total de creadores únicos: ____
   - Total de entornos: ____

   **Página: Apps (Aplicaciones)**
   - Aplicación con mayor número de sesiones de uso: ____
   - Porcentaje de aplicaciones sin descripción: ____%
   - Aplicaciones creadas en los últimos 30 días: ____

   **Página: Flows (Flujos)**
   - Flujo con mayor número de ejecuciones: ____
   - Flujos en estado de error (suspended/failed): ____
   - Distribución por tipo de flujo (compare con lo anotado en el Paso 3): ✅ / ❌

   **Página: Connectors (Conectores)**
   - Conector más utilizado en el tenant: ____
   - Conectores clasificados como **premium** en uso: ____
   - ¿Se detecta uso del conector `HTTP` (potencialmente riesgoso)? ✅ / ❌

5. En la página **Apps**, aplique el filtro **"Apps with no owner"** (Aplicaciones sin propietario) y verifique si las aplicaciones huérfanas identificadas en el Paso 3 aparecen en este filtro.

6. Tome una captura de pantalla de la página **Overview** del panel para incluirla en el plan de gobierno que elaborará en el Paso 6.

> **💡 Consejo:** Si Power BI Desktop no está disponible, el instructor puede haber publicado el panel en Power BI Service. En ese caso, acceda a [https://app.powerbi.com](https://app.powerbi.com) y busque el workspace **CoE - Contoso Industries**.

**Resultado esperado:** El panel de Power BI muestra datos reales del tenant, con visualizaciones que permiten identificar patrones de uso, aplicaciones huérfanas y conectores de riesgo. El estudiante tiene un panorama cuantitativo del estado de gobierno del tenant.

**Verificación:**
- ✅ El panel de Power BI carga datos sin errores de conexión.
- ✅ La página **Overview** muestra contadores con valores mayores a cero.
- ✅ La página **Connectors** identifica al menos un conector premium en uso.
- ✅ Se ha tomado la captura de pantalla de la página Overview.

---

### Paso 5 — Configurar los flujos de gobierno para notificaciones automáticas

**Objetivo:** Activar y configurar los flujos de gobierno del CoE Starter Kit para automatizar la comunicación con creadores de aplicaciones que no cumplen con las políticas de gobierno de Contoso Industries.

#### Instrucciones

1. En el portal de Power Automate, navegue a **Soluciones** → **Center of Excellence - Governance Components** → **Flujos de nube**.

2. Localice el flujo **GOVERNANCE - Admin | Compliance detail request v3**. Este es un flujo de tipo **Automated** (según la taxonomía de la Lección 4.1) que se activa cuando una aplicación no cumple los criterios de gobierno definidos.

3. Haga clic en el flujo para abrir su detalle. Revise su estructura:

   ```
   Trigger:    Recurrence (Scheduled) — se ejecuta semanalmente
   Acción 1:   Listar aplicaciones del CoE que no tienen descripción
   Acción 2:   Listar aplicaciones sin política de privacidad documentada
   Condición:  ¿La aplicación tiene más de 30 días de antigüedad?
     → SÍ:    Enviar correo electrónico al creador solicitando información de cumplimiento
     → NO:    Registrar en log y omitir
   Acción 3:   Actualizar registro de la aplicación en Dataverse con fecha de notificación
   ```

4. Antes de activar el flujo, personalice el mensaje de correo electrónico para Contoso Industries. Haga clic en la acción **Send an email (V2)** dentro del flujo y modifique el cuerpo del mensaje:

   ```
   Asunto: [Contoso Industries] Solicitud de información de gobierno - @{triggerOutputs()?['body/name']}

   Estimado/a @{triggerOutputs()?['body/_ownerid_value@OData.Community.Display.V1.FormattedValue']},

   Hemos detectado que la aplicación "@{triggerOutputs()?['body/name']}" 
   no cuenta con la documentación de gobierno requerida por las políticas 
   de Contoso Industries.

   Por favor, complete la siguiente información antes del @{addDays(utcNow(), 14, 'dd/MM/yyyy')}:
   - Descripción funcional de la aplicación
   - Clasificación de datos que maneja (Público / Interno / Confidencial)
   - Nombre del responsable de negocio (Business Owner)

   Para actualizar esta información, acceda al portal de creadores:
   https://make.powerapps.com

   Si tiene preguntas, contacte al equipo de gobierno en: coe-admin@contoso.com

   Equipo de Gobierno de Power Platform
   Contoso Industries
   ```

5. Guarde los cambios en el flujo y haga clic en **Activar** para habilitar el flujo.

6. Para probar el flujo sin esperar a la próxima ejecución programada, haga clic en **Ejecutar** → confirme la ejecución manual.

7. Navegue a **Historial de ejecuciones** del flujo y verifique que la ejecución manual se completó exitosamente (estado: **Correcto** en verde).

8. Adicionalmente, active el flujo **GOVERNANCE - Admin | App Archive and Clean Up v3** siguiendo el mismo procedimiento. Este flujo automatiza el archivado de aplicaciones inactivas por más de 90 días, lo que representa un ejemplo de **Cloud Flow Scheduled** de gobierno del ciclo de vida.

**Resultado esperado:** El flujo de notificaciones de cumplimiento está activado y configurado con el mensaje personalizado de Contoso Industries. La ejecución manual muestra estado **Correcto** y se han enviado correos de notificación a los creadores de aplicaciones no conformes.

**Verificación:**
- ✅ El flujo **GOVERNANCE - Admin | Compliance detail request v3** aparece en estado **Activado**.
- ✅ El historial de ejecuciones muestra al menos una ejecución con estado **Correcto**.
- ✅ El flujo **GOVERNANCE - Admin | App Archive and Clean Up v3** está activado.
- ✅ (Opcional) Se recibe un correo de notificación de prueba en el buzón del creador de una aplicación de prueba.

---

### Paso 6 — Elaborar el plan de gobierno integral de Power Platform

**Objetivo:** Consolidar todos los aprendizajes del curso en un plan de gobierno integral documentado para Contoso Industries, integrando estrategia de entornos, seguridad en Dataverse, gestión de soluciones y monitoreo continuo con CoE.

#### Instrucciones

1. Abra un nuevo documento en Microsoft Word, OneNote o el editor de su preferencia. El documento se llamará:
   ```
   Plan_Gobierno_PowerPlatform_ContosoIndustries_v1.0.docx
   ```

2. El plan debe incluir las siguientes secciones. Complete cada sección utilizando los datos recopilados en los pasos anteriores de esta práctica y en las Prácticas 1, 2 y 3:

---

**SECCIÓN 1: Estrategia de Entornos**

```
1.1 Inventario de entornos actuales
    - Entorno de Desarrollo: [nombre] — Propósito: [descripción]
    - Entorno de Producción: [nombre] — Propósito: [descripción]
    - Entorno CoE: [nombre] — Propósito: Gobierno y monitoreo
    - Entorno Default: [nombre] — Restricciones aplicadas: [DLP, acceso]

1.2 Política de creación de nuevos entornos
    - ¿Quién puede solicitar nuevos entornos? [Rol requerido]
    - Proceso de aprobación: [Flujo de aprobación en Power Automate]
    - Nomenclatura estándar: [CONTOSO-{ÁREA}-{PROPÓSITO}-{REGIÓN}]
    - Revisión periódica: [Mensual/Trimestral]
```

**SECCIÓN 2: Seguridad y Control de Acceso en Dataverse**

```
2.1 Unidades de negocio (Business Units) configuradas
    - [Lista de BUs creadas en Práctica 3]

2.2 Roles de seguridad aplicados
    - [Roles personalizados creados y sus privilegios clave]

2.3 Políticas DLP activas
    - Política de Producción: Conectores permitidos / bloqueados
    - Política de Desarrollo: Conectores permitidos / bloqueados
    - Conectores de riesgo identificados en Power BI CoE: [lista]
```

**SECCIÓN 3: Gestión del Ciclo de Vida de Soluciones (ALM)**

```
3.1 Pipeline de deployment
    Desarrollo → [Proceso de exportación] → Validación con Solution Checker
    → [Proceso de importación] → Producción

3.2 Política de versionado
    - Formato de versión: [Mayor.Menor.Parche.Build]
    - Rama de control de versiones: [Git/Azure DevOps]

3.3 Proceso de aprobación para nuevas soluciones
    - Paso 1: Solicitud del creador via [Innovation Backlog del CoE]
    - Paso 2: Revisión técnica por [Arquitecto de Power Platform]
    - Paso 3: Aprobación de negocio por [Business Owner]
    - Paso 4: Deployment por [Administrador de Power Platform]
```

**SECCIÓN 4: Monitoreo Continuo con CoE Starter Kit**

```
4.1 Componentes CoE activados
    ✅ Core — Sincronización de inventario (Scheduled, semanal)
    ✅ Governance — Notificaciones de cumplimiento (Automated)
    ⬜ Nurture — Capacitación de creadores (pendiente activación)
    ⬜ Innovation Backlog — Gestión de ideas (pendiente activación)

4.2 KPIs de gobierno (basados en datos del panel Power BI)
    - Total de aplicaciones en tenant: [dato del Paso 4]
    - % de aplicaciones con descripción: [dato del Paso 4]
    - Total de flujos activos: [dato del Paso 4]
    - Aplicaciones huérfanas identificadas: [dato del Paso 3]
    - Conectores premium en uso: [dato del Paso 4]

4.3 Cadencia de revisión
    - Revisión semanal: Flujos en error, nuevas aplicaciones sin dueño
    - Revisión mensual: KPIs del panel Power BI, cumplimiento DLP
    - Revisión trimestral: Estrategia de entornos, licencias, capacidad
```

**SECCIÓN 5: Roles y Responsabilidades de Gobierno**

```
5.1 Estructura de gobierno propuesta para Contoso Industries

    ┌─────────────────────────────────────────────────────────┐
    │              COMITÉ DE GOBIERNO DE POWER PLATFORM        │
    │                   (Revisión trimestral)                  │
    └──────────────────────────┬──────────────────────────────┘
                               │
              ┌────────────────┼────────────────┐
              ▼                ▼                ▼
    ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
    │ Administrador│  │  Arquitecto  │  │ Champion de  │
    │ de Platform  │  │  de Platform │  │ Adopción     │
    │              │  │              │  │ (por área)   │
    └──────────────┘  └──────────────┘  └──────────────┘

5.2 Responsabilidades por rol
    - Administrador de Platform: Gestión de entornos, DLP, CoE
    - Arquitecto de Platform: Revisión técnica de soluciones, ALM
    - Champion de Adopción: Capacitación, soporte a creadores, Nurture
```

3. Una vez completado el documento, compártalo con el instructor a través del método indicado (Teams, correo, SharePoint).

**Resultado esperado:** El estudiante cuenta con un plan de gobierno integral documentado que integra todos los componentes aprendidos en el curso: entornos, seguridad, ALM, CoE y roles. El documento sirve como entregable final del laboratorio.

**Verificación:**
- ✅ El documento contiene las 5 secciones del plan de gobierno.
- ✅ La Sección 4.2 (KPIs) contiene datos reales obtenidos del panel de Power BI del CoE.
- ✅ La Sección 3.3 incluye el flujo de Innovation Backlog como mecanismo de aprobación.
- ✅ El documento ha sido compartido con el instructor.

---

## Validación y pruebas

Una vez completados todos los pasos, realice las siguientes verificaciones finales para confirmar que el entorno de gobierno está operativo:

### Lista de verificación final

```powershell
# Verificación 1: Confirmar que los flujos del Core están activos
# Ejecutar en pac CLI
pac flow list --environment [ID-entorno-CoE]
```

| # | Elemento a verificar                                          | Estado esperado        | Método de verificación                          |
|---|---------------------------------------------------------------|------------------------|-------------------------------------------------|
| 1 | Soluciones CoE instaladas en entorno de gobierno              | 3 soluciones presentes | Power Apps → Soluciones                         |
| 2 | Variables de entorno configuradas                             | Sin valores vacíos     | Solución Core → Variables de entorno            |
| 3 | Flujos del Core en estado Activado                            | Verde / Activado       | Power Automate → Flujos de solución             |
| 4 | Tablas de inventario con registros                            | > 0 registros          | Power Platform Admin View (Model-driven App)    |
| 5 | Panel de Power BI carga datos del tenant                      | Datos visibles         | Power BI Desktop / Service                      |
| 6 | Flujo de notificaciones de gobierno activado                  | Activado y ejecutado   | Historial de ejecuciones del flujo              |
| 7 | Plan de gobierno documentado                                  | Documento completo     | Revisión del instructor                         |

### Prueba de extremo a extremo

Para validar el ciclo completo de gobierno, realice la siguiente prueba:

1. Cree una aplicación de lienzo de prueba **sin descripción** en el entorno de Desarrollo:
   - Nombre: `TEST-SinDescripcion-[SusIniciales]`
   - No agregue descripción ni etiquetas

2. Espere 5 minutos y luego ejecute manualmente el flujo **CORE - Sync Flows v3** para que la nueva aplicación aparezca en el inventario.

3. Verifique que la nueva aplicación aparece en la tabla **Apps** de la aplicación **Power Platform Admin View** con el campo **Description** vacío.

4. Ejecute manualmente el flujo **GOVERNANCE - Admin | Compliance detail request v3** y verifique que se genera una notificación para la nueva aplicación de prueba.

5. Revise el panel de Power BI (actualice los datos) y confirme que la nueva aplicación aparece en el conteo de **"Apps without description"**.

---

## Solución de problemas

### Problema 1: Los flujos del CoE Core no se activan o muestran errores de conexión

**Síntoma:** Al intentar activar los flujos del componente Core del CoE Starter Kit, aparece el error `"The connection is not valid"` o `"Connection reference is missing"`. Los flujos aparecen en estado **Desactivado** con un ícono de advertencia naranja.

**Causa raíz:** Las referencias de conexión (Connection References) de la solución no están vinculadas a conexiones activas del usuario actual. Esto ocurre frecuentemente cuando la solución fue importada por un usuario diferente al que intenta activar los flujos, o cuando las conexiones expiraron.

**Solución:**

1. En el portal de Power Apps, navegue a **Soluciones** → **Center of Excellence - Core Components**.
2. Haga clic en el menú **...** de la solución → **Editar** → seleccione la pestaña **Referencias de conexión**.
3. Para cada referencia de conexión que muestre estado de error (ícono rojo), haga clic en ella y seleccione **Crear nueva conexión** o **Seleccionar conexión existente** eligiendo una conexión activa de su cuenta.
4. Los conectores que típicamente necesitan reconexión son:
   ```
   - Microsoft Dataverse
   - Office 365 Users
   - Office 365 Outlook
   - Power Platform for Admins (V2)
   - HTTP with Azure AD
   ```
5. Una vez actualizadas todas las referencias de conexión, regrese a los flujos e intente activarlos nuevamente.
6. Si el problema persiste, ejecute el siguiente comando con pac CLI para reimportar la solución con actualización de conexiones:
   ```powershell
   pac solution import --path "./CenterOfExcellenceCoreComponents.zip" \
     --activate-plugins true \
     --force-overwrite true
   ```

---

### Problema 2: El panel de Power BI del CoE muestra datos vacíos o el error "No se puede conectar al origen de datos"

**Síntoma:** Al abrir el archivo `CoEDashboard.pbix` en Power BI Desktop, todas las visualizaciones muestran "No hay datos disponibles" o aparece el error `"DataSource.Error: OData: Request failed: The remote server returned an error: (401) Unauthorized"`.

**Causa raíz:** Las credenciales de autenticación configuradas en Power BI Desktop para el origen de datos de Dataverse han expirado o no corresponden a una cuenta con acceso al entorno CoE. Power BI utiliza credenciales almacenadas en caché que pueden quedar desactualizadas.

**Solución:**

1. En Power BI Desktop, navegue a **Archivo** → **Opciones y configuración** → **Configuración del origen de datos**.
2. Localice el origen de datos con la URL de su entorno CoE (formato: `https://[entorno].crm.dynamics.com`).
3. Haga clic en **Editar permisos** → **Credenciales** → **Editar**.
4. Seleccione el método de autenticación **Cuenta organizativa** y haga clic en **Iniciar sesión**. Autentíquese con la cuenta de administrador del tenant.
5. Haga clic en **Guardar** y luego en **Actualizar** en el panel principal de Power BI Desktop.
6. Si el problema persiste debido a permisos insuficientes en Dataverse, verifique que el usuario tenga el rol de seguridad **CoE - Admin** asignado en el entorno CoE:
   ```
   Power Platform Admin Center → Entornos → [CoE-Contoso] 
   → Configuración → Usuarios + Permisos → Usuarios
   → Seleccionar usuario → Administrar roles de seguridad
   → Asignar: "CoE - Admin"
   ```
7. Espere 5 minutos para que los permisos se propaguen y vuelva a intentar la actualización en Power BI Desktop.

---

## Limpieza del entorno

> **⚠️ Importante:** Dado que el entorno CoE se utilizará como referencia para el monitoreo continuo del tenant, **NO elimine** el entorno de gobierno ni las soluciones del CoE Starter Kit. Solo elimine los recursos de prueba creados durante este laboratorio.

### Recursos a eliminar

1. **Aplicación de prueba creada en la sección de Validación:**
   - En Power Apps, navegue al entorno de Desarrollo.
   - Busque la aplicación `TEST-SinDescripcion-[SusIniciales]`.
   - Haga clic en los tres puntos (**...**) → **Eliminar** → confirme.

2. **Ejecuciones de prueba en el historial de flujos** (opcional, para mantener limpio el historial):
   - En Power Automate, navegue al flujo **GOVERNANCE - Admin | Compliance detail request v3**.
   - En **Historial de ejecuciones**, las ejecuciones de prueba se eliminan automáticamente después de 28 días. No se requiere acción manual.

3. **Correos de notificación de prueba** (si se enviaron a buzones reales):
   - Notifique a los destinatarios que los correos recibidos fueron parte de una prueba de laboratorio y pueden ser eliminados.

### Verificación de limpieza

```powershell
# Verificar que la aplicación de prueba fue eliminada del inventario
# (ejecutar después de la próxima sincronización del CoE)
pac app list --environment [ID-entorno-desarrollo]
# La aplicación TEST-SinDescripcion-[SusIniciales] NO debe aparecer en la lista
```

---

## Resumen

### Logros del laboratorio

En esta práctica integradora, el estudiante de Contoso Industries ha:

1. **Explorado** la arquitectura del CoE Starter Kit, identificando los cuatro componentes (Core, Governance, Nurture, Innovation Backlog) y su relación con los tipos de flujos de Power Platform definidos en la Lección 4.1 (Cloud Flows Automated/Scheduled, Business Process Flows).

2. **Configurado** las variables de entorno del componente Core y verificado que los flujos de sincronización de inventario están activos y conectados correctamente al tenant de Contoso Industries.

3. **Analizado** el inventario del tenant mediante el panel de Power BI del CoE, obteniendo métricas cuantitativas sobre aplicaciones, flujos, conectores y creadores, e identificando aplicaciones huérfanas y conectores de riesgo.

4. **Activado y personalizado** el flujo de gobierno **GOVERNANCE - Admin | Compliance detail request v3** para automatizar las notificaciones a creadores que no cumplen con las políticas de Contoso Industries, aplicando el concepto de Cloud Flow Automated de la Lección 4.1.

5. **Elaborado** un plan de gobierno integral que consolida todos los aprendizajes del curso: estrategia de entornos, seguridad en Dataverse, ALM, monitoreo con CoE y roles de gobierno.

### Conceptos clave reforzados

| Concepto (Lección 4.1)         | Aplicación en este laboratorio                                                    |
|--------------------------------|-----------------------------------------------------------------------------------|
| Cloud Flow — Scheduled         | Flujos de sincronización del Core CoE (CORE - Sync Flows v3)                     |
| Cloud Flow — Automated         | Flujo de notificaciones de gobierno (GOVERNANCE - Compliance detail request v3)  |
| Business Process Flow          | Innovation Backlog del CoE para gestión de solicitudes de nuevas soluciones       |
| Desktop Flow (RPA)             | Identificado en el inventario del panel Power BI como tipo de flujo a gobernar   |
| Ecosistema de 1000+ conectores | Análisis de conectores premium en uso mediante el panel Power BI del CoE          |

### Recursos adicionales

| Recurso                                                     | URL                                                                                           |
|-------------------------------------------------------------|-----------------------------------------------------------------------------------------------|
| Documentación oficial del CoE Starter Kit                   | https://learn.microsoft.com/es-es/power-platform/guidance/coe/starter-kit                   |
| Repositorio GitHub del CoE Starter Kit                      | https://github.com/microsoft/coe-starter-kit                                                  |
| Guía de instalación del CoE Core                            | https://learn.microsoft.com/es-es/power-platform/guidance/coe/setup-core-components          |
| Configuración del panel Power BI del CoE                    | https://learn.microsoft.com/es-es/power-platform/guidance/coe/setup-powerbi                  |
| Mejores prácticas de gobierno de Power Platform             | https://learn.microsoft.com/es-es/power-platform/guidance/adoption/admin-best-practices      |
| Documentación de tipos de flujos en Power Automate          | https://learn.microsoft.com/es-es/power-automate/flow-types                                  |
| Guía de licenciamiento de Power Platform                    | https://learn.microsoft.com/es-es/power-platform/admin/pricing-billing-skus                  |
| Microsoft Power Platform Adoption Framework                 | https://learn.microsoft.com/es-es/power-platform/guidance/adoption/methodology               |

---

> **📌 Nota final para el instructor:** Este laboratorio está diseñado para ejecutarse con el CoE Starter Kit pre-instalado y con la sincronización inicial ya completada. El tiempo de 35 minutos asume que los estudiantes comienzan desde el Paso 2 (configuración de variables de entorno) con la solución ya importada. Si los estudiantes realizan la instalación completa desde cero, el tiempo estimado se extiende a 90-120 minutos. Se recomienda que el instructor prepare el entorno CoE antes de la sesión siguiendo la guía de instalación oficial: https://aka.ms/CoEStarterKitSetup
