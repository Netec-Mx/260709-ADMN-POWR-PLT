# Diseño de estrategia de entornos y gobierno en Power Platform

## Metadatos

| Campo            | Detalle                                      |
|------------------|----------------------------------------------|
| **Duración**     | 45 minutos                                   |
| **Complejidad**  | Media                                        |
| **Módulo**       | 1 — Estrategia de Entornos y Gobierno        |
| **Rol objetivo** | Administrador de Power Platform              |

---

## Descripción General

En esta práctica asumirás el rol de administrador de Power Platform en **Contoso Industries**, una organización ficticia que necesita estructurar su plataforma para separar el trabajo de desarrollo del ambiente productivo. Explorarás el Centro de administración de Power Platform, crearás un entorno de tipo *Sandbox* con Dataverse habilitado, lo asociarás a un grupo de seguridad de Microsoft Entra ID y configurarás una política DLP básica que restrinja los conectores no empresariales. Finalizarás revisando el panel de capacidad de almacenamiento y documentando los elementos clave de un plan de gobierno inicial, aplicando directamente los conceptos del patrón **DEV → TEST → PROD** estudiados en la lección.

---

## Objetivos de Aprendizaje

Al completar esta práctica serás capaz de:

- [ ] Explorar y describir los entornos existentes en el Centro de administración de Power Platform, identificando su tipo y configuración actual.
- [ ] Crear un entorno de tipo *Sandbox* con Dataverse habilitado y asociarlo a un grupo de seguridad de Microsoft Entra ID para controlar el acceso.
- [ ] Configurar una política de prevención de pérdida de datos (DLP) que clasifique conectores en grupos *Negocio* y *No negocio* dentro del nuevo entorno.
- [ ] Revisar el panel de capacidad de almacenamiento de Dataverse y archivos adjuntos en el Centro de administración.
- [ ] Documentar los elementos esenciales de una estrategia de gobierno inicial para Power Platform en Contoso Industries.

---

## Prerrequisitos

### Conocimientos Previos

| Área                              | Nivel requerido                                                                 |
|-----------------------------------|---------------------------------------------------------------------------------|
| Conceptos de entornos en Power Platform | Comprensión básica (tipos, propósito, relación con Dataverse)            |
| Microsoft Entra ID (Azure AD)     | Conocimiento de grupos de seguridad y su función de control de acceso           |
| Políticas DLP en Power Platform   | Familiaridad conceptual con clasificación de conectores                         |
| Centro de administración de Microsoft 365 | Navegación básica                                                      |

## Entorno de Laboratorio


### Software Requerido

| Software                              | Versión / Acceso                         | URL de acceso                                              |
|---------------------------------------|------------------------------------------|------------------------------------------------------------|
| Microsoft Edge o Google Chrome        | 120.0 o superior                         | Instalado localmente                                       |
| Power Platform Admin Center           | SaaS (sin instalación)                   | https://admin.powerplatform.microsoft.com                  |
| Microsoft Entra ID (Azure AD)         | SaaS (sin instalación)                   | https://entra.microsoft.com                                |
| Microsoft 365 Admin Center            | SaaS (sin instalación)                   | https://admin.microsoft.com                                |

### Preparación del Entorno Antes de Comenzar

Antes de iniciar los pasos del laboratorio, abre las siguientes pestañas en tu navegador:

```
Pestaña 1: https://admin.powerplatform.microsoft.com
Pestaña 2: https://entra.microsoft.com
```

Verifica que hayas iniciado sesión con la cuenta de administrador de tu tenant de práctica. Confirma que el nombre del tenant visible en la esquina superior derecha corresponde al entorno de Contoso Industries (o el tenant asignado por tu instructor).

---

## Pasos del Laboratorio

---

### Paso 1 — Crear el Grupo de Seguridad en Microsoft Entra ID

**Objetivo:** Crear el grupo de seguridad `PPA-Sandbox-Dev-Contoso` en Microsoft Entra ID que se asociará al nuevo entorno *Sandbox*. Este grupo controlará qué usuarios tienen acceso al entorno de desarrollo.

> **Nota:** Si el instructor ya creó este grupo previamente, omite este paso y toma nota del nombre exacto del grupo para usarlo en el Paso 2.

#### Instrucciones

1. Navega a la pestaña del **Portal de Microsoft Entra ID**: `https://entra.microsoft.com`.

2. En el panel de navegación izquierdo, expande **Identidad** → **Grupos** → haz clic en **Todos los grupos**.

3. Haz clic en el botón **+ Nuevo grupo** en la barra de herramientas superior.

4. Completa el formulario con los siguientes valores:

   | Campo                    | Valor                                          |
   |--------------------------|------------------------------------------------|
   | **Tipo de grupo**        | Seguridad                                      |
   | **Nombre del grupo**     | `PPA-Sandbox-Dev-Contoso`                      |
   | **Descripción**          | `Grupo de acceso al entorno Sandbox de desarrollo para Contoso Industries` |
   | **Tipo de pertenencia**  | Asignado                                       |
   | **Propietarios**         | Tu cuenta de administrador                     |
   | **Miembros**             | Tu cuenta de administrador (agrégala como miembro inicial) |

5. Haz clic en **Crear**.

6. Una vez creado, abre el grupo recién creado y copia el valor del campo **Id. de objeto** — lo necesitarás como referencia en pasos posteriores.

   ```
   Ejemplo de Id. de objeto: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
   ```

#### Resultado Esperado

El grupo `PPA-Sandbox-Dev-Contoso` aparece en la lista de **Todos los grupos** con tipo **Seguridad** y al menos un miembro (tu cuenta de administrador).

#### Verificación

- En la lista de grupos, filtra por el nombre `PPA-Sandbox-Dev-Contoso` y confirma que el tipo de grupo es **Seguridad** (no Microsoft 365).
- Abre el grupo y verifica que la pestaña **Miembros** muestre al menos una cuenta.

---

### Paso 2 — Explorar los Entornos Existentes en el Centro de Administración

**Objetivo:** Familiarizarse con la vista de entornos del Centro de administración de Power Platform, identificar el entorno predeterminado y comprender su configuración actual antes de crear uno nuevo.

#### Instrucciones

1. Navega a la pestaña del **Centro de administración de Power Platform**: `https://admin.powerplatform.microsoft.com`.

2. En el panel de navegación izquierdo, haz clic en **Entornos**.

3. Observa la lista de entornos disponibles. Identifica el entorno de tipo **Predeterminado** (Default) — generalmente tiene el nombre del tenant seguido de `(default)`.

4. Haz clic sobre el entorno **Predeterminado** para abrir su panel de detalles.

5. Examina y registra la siguiente información en tu documento de estrategia (ver Paso 6):

   | Campo a registrar           | Descripción                                              |
   |-----------------------------|----------------------------------------------------------|
   | **Nombre del entorno**      | Nombre visible del entorno predeterminado                |
   | **Región**                  | Región de Azure donde está hospedado                     |
   | **Tipo**                    | Debe indicar "Predeterminado"                            |
   | **Estado**                  | Activo / Inactivo                                        |
   | **Dataverse habilitado**    | Sí / No                                                  |
   | **Grupo de seguridad**      | Observa si tiene uno asignado o si está abierto a todos  |
   | **URL del entorno**         | URL única del entorno                                    |

6. Regresa a la lista de entornos haciendo clic en **Entornos** en el panel izquierdo.

7. Revisa si existen otros entornos (Producción, Sandbox, etc.) y anota cuántos hay en total.

#### Resultado Esperado

Visualizas la lista completa de entornos del tenant. El entorno predeterminado está identificado y has registrado sus atributos principales. Observas que el entorno predeterminado probablemente **no tiene** un grupo de seguridad asignado, lo que significa que todos los usuarios con licencia tienen acceso a él — una situación que la estrategia de gobierno deberá abordar.

#### Verificación

- Confirma que puedes distinguir visualmente el entorno **Predeterminado** de los demás tipos por el indicador de tipo en la columna correspondiente.
- Verifica que hayas registrado al menos los 7 campos de la tabla anterior.

---

### Paso 3 — Crear el Entorno Sandbox con Dataverse Habilitado

**Objetivo:** Crear un nuevo entorno de tipo *Sandbox* denominado `Dev-Soluciones-Contoso`, habilitando Dataverse y asociando el grupo de seguridad creado en el Paso 1. Este entorno representará el nivel de desarrollo en la estrategia DEV → TEST → PROD de Contoso Industries.

#### Instrucciones

1. En el Centro de administración de Power Platform, navega a **Entornos** y haz clic en **+ Nuevo** en la barra de herramientas superior.

2. Se abrirá el panel **Nuevo entorno** en el lado derecho. Completa los campos con los siguientes valores:

   | Campo                    | Valor                                                     |
   |--------------------------|-----------------------------------------------------------|
   | **Nombre**               | `Dev-Soluciones-Contoso`                                  |
   | **Región**               | Selecciona la región más cercana a tu ubicación (ej.: *Brazil South* para América del Sur, *East US* para Norteamérica) |
   | **Tipo**                 | Espacio aislado (*Sandbox*)                               |
   | **Propósito**            | `Entorno de desarrollo activo para soluciones de Contoso Industries` |

   > **Importante:** La región seleccionada es **permanente** y no puede cambiarse después de la creación. Elige con cuidado según los requisitos de residencia de datos de tu organización.

3. Activa el interruptor **Agregar una base de datos de Dataverse a este entorno** → cambia a **Sí**.

4. Aparecerán campos adicionales para la configuración de Dataverse. Completa:

   | Campo                    | Valor                                                     |
   |--------------------------|-----------------------------------------------------------|
   | **Idioma**               | Español (España) o Español (México) según corresponda     |
   | **Moneda**               | Selecciona la moneda de tu región (USD, EUR, MXN, etc.)   |
   | **Habilitar aplicaciones de Dynamics 365** | No (para este laboratorio no es necesario) |
   | **Implementar aplicaciones y datos de ejemplo** | No                               |

5. Haz clic en **Siguiente**.

6. En la sección **Grupo de seguridad**, haz clic en el campo de búsqueda y escribe `PPA-Sandbox-Dev-Contoso`. Selecciona el grupo creado en el Paso 1.

   > **Nota técnica:** Al asignar un grupo de seguridad, solo los usuarios miembros de ese grupo podrán ver y acceder a este entorno. Esto implementa el principio de mínimo privilegio desde el momento de la creación.

7. Revisa el resumen de configuración y haz clic en **Guardar**.

8. El sistema comenzará a aprovisionar el entorno. Observarás el estado **Preparando** en la lista de entornos. Este proceso puede tomar entre **3 y 7 minutos** para completarse.

   > **Mientras esperas:** Avanza al Paso 4 para revisar el panel de capacidad, y regresa a verificar el estado del entorno antes de continuar con el Paso 5.

#### Resultado Esperado

El entorno `Dev-Soluciones-Contoso` aparece en la lista de entornos con:
- **Tipo:** Espacio aislado (Sandbox)
- **Estado:** Listo (después del aprovisionamiento)
- **Base de datos:** Dataverse habilitado
- **Grupo de seguridad:** `PPA-Sandbox-Dev-Contoso`

#### Verificación

Una vez completado el aprovisionamiento:
- Haz clic sobre el entorno `Dev-Soluciones-Contoso` y verifica que la sección **Detalles** muestre todos los atributos configurados.
- Confirma que en la sección **Grupo de seguridad** aparece `PPA-Sandbox-Dev-Contoso` y no el valor "Ninguno".
- Verifica que la sección **Base de datos** muestre información de Dataverse (versión, capacidad utilizada inicial).

---

### Paso 4 — Revisar el Panel de Capacidad de Almacenamiento

**Objetivo:** Explorar el panel de capacidad del Centro de administración para entender el consumo actual de almacenamiento de Dataverse y archivos adjuntos en el tenant, como parte del monitoreo de gobierno.

#### Instrucciones

1. En el Centro de administración de Power Platform, en el panel de navegación izquierdo, haz clic en **Recursos** → **Capacidad**.

2. Se abre el panel **Resumen de capacidad**. Examina las tres categorías principales:

   | Categoría de almacenamiento | Descripción                                                          |
   |-----------------------------|----------------------------------------------------------------------|
   | **Base de datos**           | Almacenamiento relacional de Dataverse (tablas, registros)           |
   | **Archivos**                | Archivos adjuntos y datos de tipo archivo en Dataverse               |
   | **Registro**                | Datos de auditoría y registros de seguimiento                        |

3. Registra los valores actuales en tu documento de estrategia:

   ```
   Capacidad total de base de datos    : ___ GB
   Capacidad usada de base de datos    : ___ GB
   Capacidad disponible                : ___ GB
   
   Capacidad total de archivos         : ___ GB
   Capacidad usada de archivos         : ___ GB
   
   Número de entornos con Dataverse    : ___
   ```

4. Desplázate hacia abajo para ver la tabla **Capacidad por entorno**. Identifica el entorno `Dev-Soluciones-Contoso` recién creado (puede aparecer con valores iniciales mínimos).

5. Observa el entorno **Predeterminado** y verifica cuánta capacidad está consumiendo actualmente.

6. Haz clic en la pestaña **Complementos** (si está disponible) para revisar si el tenant tiene capacidad adicional adquirida.

7. Regresa al panel principal de **Entornos** para verificar si el entorno `Dev-Soluciones-Contoso` ya cambió su estado a **Listo**.

#### Resultado Esperado

Visualizas el desglose completo de capacidad de almacenamiento del tenant, con los valores de base de datos, archivos y registro. El nuevo entorno `Dev-Soluciones-Contoso` aparece en la tabla de capacidad por entorno con una asignación inicial.

#### Verificación

- Confirma que el total de capacidad disponible es mayor a **0 GB** (el tenant tiene capacidad suficiente para operar).
- Verifica que el entorno `Dev-Soluciones-Contoso` aparece en la lista de entornos con Dataverse.
- Si la capacidad disponible es inferior a 500 MB, notifica al instructor antes de continuar.

---

### Paso 5 — Configurar una Política DLP para el Entorno Sandbox

**Objetivo:** Crear una política de prevención de pérdida de datos (DLP) que se aplique específicamente al entorno `Dev-Soluciones-Contoso`, clasificando los conectores empresariales aprobados en el grupo *Negocio* y bloqueando los conectores no empresariales.

#### Instrucciones

1. En el Centro de administración de Power Platform, en el panel de navegación izquierdo, haz clic en **Políticas** → **Políticas de datos**.

2. Haz clic en **+ Nueva política** en la barra de herramientas superior.

3. En el campo **Nombre de la política**, escribe:
   ```
   DLP-Dev-Contoso-Conectores-Empresariales
   ```

4. Haz clic en **Siguiente**.

5. En la pantalla **Asignar conectores**, verás tres grupos:
   - **Negocio** (*Business*) — Conectores aprobados para uso empresarial
   - **No negocio** (*Non-business*) — Conectores permitidos pero aislados de los de negocio
   - **Bloqueado** (*Blocked*) — Conectores completamente deshabilitados

6. Mueve los siguientes conectores al grupo **Negocio** buscándolos en la lista y arrastrándolos o usando el botón **Mover a negocio**:

   | Conector                        | Justificación                                      |
   |---------------------------------|----------------------------------------------------|
   | Microsoft Dataverse             | Almacenamiento de datos empresariales principal    |
   | SharePoint                      | Gestión documental corporativa                     |
   | Microsoft Teams                 | Comunicación y colaboración interna                |
   | Office 365 Outlook              | Correo electrónico corporativo                     |
   | Office 365 Users                | Directorio de usuarios organizacionales            |
   | Approvals                       | Flujos de aprobación empresariales                 |

7. Mueve los siguientes conectores al grupo **Bloqueado**:

   | Conector                        | Justificación de bloqueo                           |
   |---------------------------------|----------------------------------------------------|
   | Twitter / X                     | Red social externa sin uso empresarial aprobado    |
   | Gmail                           | Correo externo no corporativo                      |
   | Dropbox                         | Almacenamiento en la nube externo no aprobado      |

   > **Nota:** Los conectores no clasificados explícitamente permanecen en el grupo **No negocio** por defecto. Esto significa que pueden usarse en aplicaciones, pero no pueden combinarse en el mismo flujo o aplicación con conectores del grupo **Negocio**.

8. Haz clic en **Siguiente**.

9. En la pantalla **Definir ámbito**, selecciona la opción **Agregar varios entornos** y luego elige **Aplicar a entornos específicos**.

10. En el campo de búsqueda, escribe `Dev-Soluciones-Contoso` y selecciona el entorno. Haz clic en **Agregar a política**.

    > **Importante:** No selecciones "Todos los entornos" ni "Todos excepto seleccionados", ya que esto afectaría entornos fuera del alcance de esta práctica.

11. Confirma que el entorno `Dev-Soluciones-Contoso` aparece en la lista de entornos asignados a esta política.

12. Haz clic en **Siguiente** y revisa el resumen de la política:

    ```
    Nombre de la política  : DLP-Dev-Contoso-Conectores-Empresariales
    Ámbito                 : Dev-Soluciones-Contoso (1 entorno)
    Conectores en Negocio  : 6 (Dataverse, SharePoint, Teams, Outlook, Users, Approvals)
    Conectores Bloqueados  : 3 (Twitter/X, Gmail, Dropbox)
    Conectores No negocio  : Resto de conectores disponibles
    ```

13. Haz clic en **Crear política**.

#### Resultado Esperado

La política `DLP-Dev-Contoso-Conectores-Empresariales` aparece en la lista de **Políticas de datos** con estado **Activo** y el ámbito muestra `1 entorno`. Los conectores empresariales críticos están en el grupo *Negocio* y los conectores no aprobados están en el grupo *Bloqueado*.

#### Verificación

- En la lista de **Políticas de datos**, haz clic sobre `DLP-Dev-Contoso-Conectores-Empresariales` y verifica:
  - La pestaña **Conectores** muestra al menos 6 conectores en el grupo **Negocio**.
  - La pestaña **Ámbito** muestra `Dev-Soluciones-Contoso` como único entorno asignado.
  - El estado de la política es **Activo**.
- Confirma que la política **no** está asignada al entorno **Predeterminado** ni a otros entornos del tenant.

---

### Paso 6 — Documentar la Estrategia de Gobierno Inicial

**Objetivo:** Elaborar un documento de estrategia de gobierno básico que consolide las decisiones tomadas durante la práctica, identificando roles, tipos de entornos recomendados y políticas de acceso para Contoso Industries.

#### Instrucciones

1. Abre un editor de texto (puede ser el Bloc de notas, Microsoft Word o un documento en SharePoint/OneDrive) y crea un documento titulado:

   ```
   Estrategia de Gobierno de Power Platform — Contoso Industries
   Versión 1.0 | Fecha: [fecha actual]
   Autor: [tu nombre]
   ```

2. Completa las siguientes secciones del documento utilizando la información recopilada durante los pasos anteriores:

---

**Sección 1: Inventario de Entornos**

Documenta los entornos identificados y creados, siguiendo el patrón DEV → TEST → PROD:

| Entorno                   | Tipo          | Dataverse | Grupo de Seguridad              | Propósito                                        |
|---------------------------|---------------|-----------|---------------------------------|--------------------------------------------------|
| `[Nombre del Default]`    | Predeterminado| Sí        | Ninguno (pendiente de asignar)  | Experimentación personal controlada              |
| `Dev-Soluciones-Contoso`  | Sandbox       | Sí        | PPA-Sandbox-Dev-Contoso         | Desarrollo activo de aplicaciones y flujos       |
| `Test-QA-Contoso` *(futuro)* | Sandbox    | Sí        | PPA-QA-Contoso *(por crear)*    | Pruebas de aceptación de usuario (UAT)           |
| `Prod-Soluciones-Contoso` *(futuro)* | Producción | Sí | PPA-Prod-Contoso *(por crear)*  | Aplicaciones en uso productivo                   |

---

**Sección 2: Roles de Administración**

```
Rol                              Responsabilidad Principal
─────────────────────────────────────────────────────────────────────
Administrador Global             Gestión de tenant, licenciamiento y 
                                 configuración de nivel superior.

Administrador de Power Platform  Creación y configuración de entornos,
                                 políticas DLP y monitoreo de capacidad.

Administrador de Entorno         Gestión de usuarios y roles dentro de 
                                 un entorno específico.

Maker (Creador)                  Desarrollo de aplicaciones y flujos 
                                 dentro de los entornos autorizados.

Usuario Final                    Consumo de aplicaciones publicadas 
                                 en entornos de producción.
```

---

**Sección 3: Políticas DLP Implementadas**

| Política                                    | Entorno Aplicado          | Conectores Negocio | Conectores Bloqueados |
|---------------------------------------------|---------------------------|--------------------|-----------------------|
| `DLP-Dev-Contoso-Conectores-Empresariales`  | Dev-Soluciones-Contoso    | 6                  | 3                     |
| *(Pendiente)* DLP-Prod-Contoso              | Prod-Soluciones-Contoso   | Por definir        | Por definir           |

---

**Sección 4: Principios de Gobierno Adoptados**

Registra los siguientes principios como parte de la estrategia:

```
1. AISLAMIENTO: Cada fase del ciclo de vida (Dev, Test, Prod) opera 
   en entornos separados con acceso controlado por grupos de seguridad.

2. MÍNIMO PRIVILEGIO: Los usuarios solo tienen acceso a los entornos 
   que necesitan para su función. El acceso se gestiona mediante 
   grupos de seguridad de Microsoft Entra ID.

3. CONTROL DE CONECTORES: Las políticas DLP se aplican por entorno 
   para garantizar que solo se usen conectores aprobados por el negocio.

4. RESIDENCIA DE DATOS: Todos los entornos de Contoso Industries se 
   crean en la región [región seleccionada] para cumplir con los 
   requisitos de soberanía de datos.

5. MONITOREO DE CAPACIDAD: La capacidad de Dataverse se revisa 
   mensualmente desde el Centro de administración para anticipar 
   necesidades de escalamiento.
```

---

**Sección 5: Próximos Pasos Recomendados**

```
□ Crear entornos Test-QA-Contoso y Prod-Soluciones-Contoso
□ Crear grupos de seguridad para cada entorno adicional
□ Configurar políticas DLP para entornos de Test y Prod
□ Asignar grupo de seguridad al entorno Predeterminado para restringir acceso
□ Implementar el CoE Starter Kit para monitoreo avanzado de adopción
□ Definir proceso formal de ALM (Dev → Test → Prod) con pipelines
□ Establecer proceso de revisión y aprobación para nuevos entornos
```

3. Guarda el documento. Si usas OneDrive o SharePoint, comparte el enlace con tu instructor.

#### Resultado Esperado

Un documento estructurado de estrategia de gobierno que refleja las decisiones tomadas durante la práctica, con las cinco secciones completadas y los valores reales del tenant registrados.

#### Verificación

- El documento contiene al menos las 5 secciones especificadas.
- La tabla de entornos incluye el entorno `Dev-Soluciones-Contoso` con los valores correctos configurados durante la práctica.
- Los principios de gobierno están registrados y reflejan las configuraciones realizadas.

---

## Validación y Pruebas Finales

Antes de considerar la práctica completada, realiza las siguientes verificaciones en el Centro de administración:

### Lista de Verificación Final

| # | Elemento a verificar                                                                 | Estado |
|---|--------------------------------------------------------------------------------------|--------|
| 1 | El grupo `PPA-Sandbox-Dev-Contoso` existe en Microsoft Entra ID con tipo **Seguridad** | ☐ |
| 2 | El entorno `Dev-Soluciones-Contoso` existe con tipo **Sandbox** y estado **Listo**    | ☐ |
| 3 | El entorno `Dev-Soluciones-Contoso` tiene **Dataverse habilitado**                    | ☐ |
| 4 | El entorno `Dev-Soluciones-Contoso` tiene el grupo `PPA-Sandbox-Dev-Contoso` asignado | ☐ |
| 5 | La política `DLP-Dev-Contoso-Conectores-Empresariales` existe con estado **Activo**   | ☐ |
| 6 | La política DLP está asignada **únicamente** al entorno `Dev-Soluciones-Contoso`      | ☐ |
| 7 | La política DLP tiene al menos 6 conectores en el grupo **Negocio**                   | ☐ |
| 8 | La política DLP tiene al menos 3 conectores en el grupo **Bloqueado**                 | ☐ |
| 9 | El panel de capacidad muestra valores de almacenamiento registrados en el documento   | ☐ |
| 10 | El documento de estrategia de gobierno está completado con las 5 secciones           | ☐ |

### Prueba de Acceso al Entorno con Grupo de Seguridad

Para validar que el grupo de seguridad funciona correctamente:

1. En el Centro de administración, navega a **Entornos** → haz clic en `Dev-Soluciones-Contoso`.
2. Haz clic en **Ver todo** en la sección **Acceso** → **Usuarios**.
3. Verifica que el listado de usuarios autorizados corresponde a los miembros del grupo `PPA-Sandbox-Dev-Contoso`.
4. Confirma que el entorno **no** es accesible para usuarios que no son miembros del grupo (esto se puede verificar conceptualmente revisando que el grupo tiene pertenencia limitada).

---

## Solución de Problemas

### Problema 1: El entorno `Dev-Soluciones-Contoso` permanece en estado "Preparando" por más de 10 minutos

**Síntoma:** Después de crear el entorno, el estado no cambia de **Preparando** a **Listo** tras más de 10 minutos de espera. El entorno aparece en la lista pero no es funcional.

**Causa probable:** El aprovisionamiento de Dataverse puede demorarse más de lo esperado si el tenant tiene capacidad de almacenamiento limitada, si hay una degradación temporal del servicio en la región seleccionada, o si el tenant está procesando múltiples operaciones simultáneas.

**Solución:**

1. Verifica el estado del servicio de Power Platform en `https://admin.microsoft.com` → **Estado del servicio** → busca alertas para **Power Platform** o **Dataverse**.
2. Navega al panel de **Capacidad** (`Recursos → Capacidad`) y confirma que hay al menos **1 GB disponible** de almacenamiento de base de datos.
3. Si el estado del servicio es normal y hay capacidad disponible, espera 5 minutos adicionales y actualiza la página del Centro de administración con `F5`.
4. Si el problema persiste después de 20 minutos, navega a **Entornos**, selecciona el entorno en estado **Preparando** y verifica si hay un mensaje de error en el panel de detalles. Toma nota del mensaje y notifica al instructor.
5. Como último recurso, elimina el entorno fallido y recréalo seleccionando una región diferente (si la región original muestra alertas de servicio).

---

### Problema 2: La política DLP no aparece como "Activa" o no se puede asignar al entorno Sandbox

**Síntoma:** Al crear la política `DLP-Dev-Contoso-Conectores-Empresariales`, el sistema muestra un error al intentar asignarla al entorno `Dev-Soluciones-Contoso`, o la política se crea pero aparece con estado **Inactivo** o no se visualiza en la lista de políticas de datos.

**Causa probable:** Existen dos causas frecuentes: (a) el entorno `Dev-Soluciones-Contoso` aún no completó su aprovisionamiento cuando se intentó asignar la política DLP — las políticas DLP solo pueden aplicarse a entornos en estado **Listo**; o (b) la cuenta de administrador no tiene permisos suficientes para crear políticas DLP a nivel de entorno (se requiere el rol **Administrador de Power Platform** o **Administrador Global**).

**Solución:**

1. **Verificar el estado del entorno:** Navega a **Entornos** y confirma que `Dev-Soluciones-Contoso` muestra estado **Listo** antes de intentar asignar la política DLP. Si aún está en **Preparando**, espera a que complete el aprovisionamiento.

2. **Verificar permisos de la cuenta:**
   - Navega a `https://admin.microsoft.com` → **Usuarios** → **Usuarios activos**.
   - Busca tu cuenta de administrador y verifica que tiene asignado el rol **Administrador de Power Platform** o **Administrador Global**.
   - Si el rol no está asignado, haz clic en **Administrar roles** y agrega el rol correspondiente.

3. **Recrear la política si es necesario:**
   - Si la política existe pero tiene estado incorrecto, elimínala desde **Políticas → Políticas de datos** → selecciona la política → **Eliminar**.
   - Espera 2 minutos y vuelve a crear la política desde el principio siguiendo el Paso 5.

4. **Verificar conflictos con políticas existentes:** Si ya existe una política DLP asignada a "Todos los entornos" en el tenant, esta puede entrar en conflicto. Identifica esa política, verifica su configuración y consulta con el instructor antes de modificarla.

---

## Limpieza del Entorno

> **Advertencia:** Los recursos creados en esta práctica son necesarios para las **Prácticas 2, 3 y 4** del curso. **No elimines** el entorno `Dev-Soluciones-Contoso` ni la política DLP si planeas continuar con las prácticas siguientes. Solo realiza la limpieza si el instructor lo indica explícitamente o si estás trabajando en un tenant de práctica temporal.

### Limpieza Condicional (solo si el instructor lo indica)

Si necesitas limpiar los recursos creados:

**1. Eliminar la política DLP:**
```
Centro de administración → Políticas → Políticas de datos
→ Seleccionar "DLP-Dev-Contoso-Conectores-Empresariales"
→ Clic en "Eliminar" → Confirmar eliminación
```

**2. Eliminar el entorno Sandbox:**
```
Centro de administración → Entornos
→ Seleccionar "Dev-Soluciones-Contoso"
→ Clic en "Eliminar" en la barra de herramientas
→ Escribir el nombre del entorno para confirmar → Confirmar eliminación
```

> **Nota:** La eliminación de un entorno con Dataverse es **irreversible** y elimina todos los datos, aplicaciones y flujos contenidos en él. El proceso puede tomar varios minutos.

**3. Eliminar el grupo de seguridad (opcional):**
```
Portal de Microsoft Entra ID → Identidad → Grupos → Todos los grupos
→ Buscar "PPA-Sandbox-Dev-Contoso"
→ Seleccionar el grupo → Clic en "Eliminar" → Confirmar
```

**4. Conservar el documento de estrategia de gobierno** — este documento tiene valor independientemente de los recursos del tenant y debe guardarse como referencia para prácticas futuras.

---

## Resumen

En esta práctica implementaste los fundamentos de una estrategia de entornos y gobierno en Power Platform para Contoso Industries. Los logros principales incluyen:

- **Creaste el grupo de seguridad** `PPA-Sandbox-Dev-Contoso` en Microsoft Entra ID, estableciendo el mecanismo de control de acceso basado en identidad para el entorno de desarrollo.
- **Exploraste el entorno predeterminado** e identificaste su configuración actual, reconociendo la necesidad de aplicar políticas de gobierno para evitar el uso no controlado.
- **Creaste el entorno Sandbox** `Dev-Soluciones-Contoso` con Dataverse habilitado, aplicando el patrón DEV → TEST → PROD y asignando el grupo de seguridad desde el momento de la creación.
- **Revisaste el panel de capacidad** de almacenamiento, comprendiendo cómo monitorear el consumo de Dataverse como parte de las responsabilidades de administración.
- **Configuraste la política DLP** `DLP-Dev-Contoso-Conectores-Empresariales`, clasificando conectores en grupos *Negocio* y *Bloqueado* para garantizar que solo se utilicen conectores aprobados en el entorno de desarrollo.
- **Documentaste la estrategia de gobierno inicial**, consolidando roles, tipos de entornos, políticas y principios de administración en un artefacto reutilizable.

### Conceptos Clave Aplicados

| Concepto                          | Aplicación en la Práctica                                                      |
|-----------------------------------|--------------------------------------------------------------------------------|
| Tipos de entornos                 | Creación de entorno Sandbox como parte del patrón DEV → TEST → PROD            |
| Grupos de seguridad de Entra ID   | Asignados al entorno para implementar el principio de mínimo privilegio        |
| Dataverse                         | Habilitado en el entorno Sandbox para soporte de soluciones empresariales      |
| Políticas DLP                     | Configuradas por entorno para controlar el uso de conectores                   |
| Panel de capacidad                | Revisado como parte del monitoreo de gobierno de almacenamiento                |
| Documento de estrategia           | Elaborado como artefacto de gobierno reutilizable para Contoso Industries      |

### Recursos Adicionales

| Recurso                                                                 | URL                                                                                                              |
|-------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------|
| Documentación oficial: Información general sobre entornos               | https://learn.microsoft.com/es-es/power-platform/admin/environments-overview                                     |
| Documentación oficial: Crear y administrar entornos                     | https://learn.microsoft.com/es-es/power-platform/admin/create-environment                                        |
| Documentación oficial: Entornos Administrados                           | https://learn.microsoft.com/es-es/power-platform/admin/managed-environment-overview                              |
| Documentación oficial: Crear una política DLP                           | https://learn.microsoft.com/es-es/power-platform/admin/create-dlp-policy                                         |
| Guía de adopción: Estrategia de entornos                                | https://learn.microsoft.com/es-es/power-platform/guidance/adoption/environment-strategy                          |
| Documentación oficial: Residencia de datos en Power Platform            | https://learn.microsoft.com/es-es/power-platform/admin/regions-overview                                          |
| Guía de adopción: Marco de gobierno de Power Platform                   | https://learn.microsoft.com/es-es/power-platform/guidance/adoption/admin-best-practices                          |

---

*Fin de la Práctica 1 — Continúa con la **Práctica 2: Implementación del Marco de Gobierno con el CoE Starter Kit**.*
