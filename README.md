[![Issues][issues-shield]][issues-url]
[![LinkedIn][linkedin-shield]][linkedin-url]

<!-- Intro -->
<br />
<div align="center" style="text-align:center;">
  <h1 style="font-size:40px; font-bload"><b style="color:#ec4b42">Oracle AI</b> Database Private Agent Factory</h1>

  <p>Laboratorio guiado: despliegue de Agent Factory sobre Oracle Cloud con Autonomous AI Database, OCI GenAI y un flujo conversacional basado en SQL</p>

  <a style="font-size:large;" href="https://objectstorage.us-chicago-1.oraclecloud.com/n/axzs95biupm2/b/labs/o/lab.zip">📄 Descargar Plantilla »</a>
  <br/>
  <a href="https://github.com/jganggini/oracle-ai-data-platform-workbench/issues">💣 Report Bug</a>
  ·
  <a href="https://github.com/jganggini/oracle-ai-data-platform-workbench/pulls">🚀 Request Feature</a>
    
  <div align="center">
  <a href="https://www.youtube.com/watch?v=zIxA0ebo2v4&list=PLMUWTQHw13gZlzbOKtsJwXOGizrOCsOkx">
    <img src="https://img.shields.io/badge/YouTube-Ver%20Tutorial-red?style=for-the-badge&logo=youtube" alt="Video Tutorial"/>
  </a>
  </div>

</div>

---

## 📋 Descripción del Proyecto

Este laboratorio muestra cómo desplegar **Oracle AI Database Private Agent Factory** desde **Oracle Cloud Marketplace**, conectarlo a una **Oracle Autonomous AI Database**, configurar un modelo de **OCI Generative AI**, registrar una fuente de datos y adaptar un flujo de ejemplo para responder preguntas en lenguaje natural a partir de una consulta SQL.

### ¿Qué aprenderás?

- Buscar y lanzar `Oracle AI Database Private Agent Factory` desde Marketplace
- Crear una `VCN` con conectividad a internet y abrir los puertos requeridos
- Aprovisionar la VM del stack usando `Resource Manager`
- Descargar y usar el `wallet` de `Autonomous AI Database`
- Crear el usuario de instalación para Agent Factory
- Configurar `OCI GenAI` como proveedor LLM dentro de Agent Factory
- Registrar una base de datos como `data source`
- Importar y ajustar un flujo tipo `Intent-Based SQL Dispatcher`
- Validar preguntas conversacionales contra la tabla `app_aidp.interactions_analysis`

---

## 🛠️ Arquitectura del Proyecto

```text
┌─────────────────────────────┐
│ Oracle Cloud Marketplace    │
│ Agent Factory Stack         │
└──────────────┬──────────────┘
               │
               v
┌─────────────────────────────┐
│ OCI Resource Manager        │
│ VM + Private Agent Factory  │
└───────┬─────────────────────┘
        │
        ├───────────────┐
        │               │
        v               v
┌───────────────┐   ┌────────────────────┐
│ OCI GenAI     │   │ Autonomous AI DB   │
│ Gemini / LLM  │   │ Wallet + schemas   │
└───────┬───────┘   └─────────┬──────────┘
        │                     │
        └──────────────┬──────┘
                       v
            ┌──────────────────────┐
            │ Agent Builder        │
            │ SQL Dispatcher Flow  │
            └──────────────────────┘
```

---

## 🔧 Prerrequisitos

### 1. Cuenta de Oracle Cloud Infrastructure

- Acceso a `Marketplace`, `Networking`, `Resource Manager`, `Autonomous AI Database` e `Identity`
- Un `Compartment` disponible para el laboratorio, por ejemplo `ora26ai`

### 2. Oracle Autonomous AI Database disponible

- Una base `Autonomous AI Database` ya aprovisionada
- Posibilidad de descargar el `wallet`
- Acceso a `Database Actions`

### 3. Datos de negocio ya cargados

Este laboratorio se enfoca en **Agent Factory**. Se asume que la base ya contiene el esquema de datos que consultarás desde el agente. En las capturas se utiliza la tabla:

```sql
app_aidp.interactions_analysis
```

### 4. Archivos de apoyo del repositorio

- `files/Wallet_ora26ainame.zip`: wallet de ejemplo para la conexión a la base
- `files/oracle26ai.txt`: script base para crear el usuario de instalación de Agent Factory
- `files/oci/config`: plantilla de configuración OCI
- `files/oci/key.pem`: clave privada para autenticación con OCI GenAI
- `files/oci/key_public.pem`: clave pública original
- `files/oci/key_agent_factoru_public.pem`: clave pública usada al crear la VM del stack

### 5. Consideraciones importantes

- Usa los archivos del repositorio como **plantilla**; reemplaza contraseñas, rutas, OCID y valores sensibles por los tuyos
- No reutilices en producción claves ni passwords de ejemplo
- Si tu navegador o antivirus muestra advertencias de certificado para la IP pública de la VM, valida con tu política interna antes de continuar

---

## 📚 Laboratorio Paso a Paso

> Este manual está pensado para usuarios que ejecutarán el laboratorio manualmente en OCI.

---

### Paso 1: Buscar la aplicación en Marketplace

**Step 1**

En la consola de Oracle Cloud, abre el buscador global y selecciona `All Applications`.

![img01](./img/img01.png)

**Step 2**

Busca `Agent Factory` y abre la aplicación `Oracle AI Database Private Agent Factory`.

![img02](./img/img02.png)

**Step 3**

Dentro de la ficha de la aplicación, confirma que el stack esté disponible y haz clic en `Launch Stack`.

![img03](./img/img03.png)

**Step 4**

En `Launch Stack`, selecciona el compartment donde vas a desplegar la solución.

![img04](./img/img04.png)

**Step 5**

Acepta los términos del publicador y pulsa `Launch Stack`.

![img05](./img/img05.png)

**Step 6**

En `Create stack`, revisa la información del stack y haz clic en `Next`.

![img06](./img/img06.png)

---

### Paso 2: Crear la red base y abrir puertos

**Step 7**

Busca `vcn` en la consola y entra en `Virtual cloud networks`.

![img07](./img/img07.png)

**Step 8**

En el compartment del laboratorio, abre `Actions` y elige `Start VCN Wizard`.

![img08](./img/img08.png)

**Step 9**

Selecciona la opción `Create VCN with Internet Connectivity` y continúa con `Start VCN Wizard`.

![img09](./img/img09.png)

**Step 10**

Configura la red con valores como estos: `VCN name: vcn_name`, `CIDR: 10.0.0.0/16`, `Public subnet: 10.0.0.0/24` y `Private subnet: 10.0.10.0/24`; después haz clic en `Next`.

![img10](./img/img10.png)

**Step 11**

Revisa el resumen de la VCN y pulsa `Create`.

![img11](./img/img11.png)

**Step 12**

Espera a que termine la creación de recursos y luego entra con `View VCN`.

![img12](./img/img12.png)

**Step 13**

Dentro de la VCN, abre la pestaña `Security`.

![img13](./img/img13.png)

**Step 14**

Abre la `Default Security List for vcn_name`.

![img14](./img/img14.png)

**Step 15**

En la security list, entra a la pestaña `Security rules`.

![img15](./img/img15.png)

**Step 16**

Pulsa `Add Ingress Rules`.

![img16](./img/img16.png)

**Step 17**

Agrega dos reglas `TCP` desde `0.0.0.0/0`: una para el puerto `8080` con descripción `Agent Factory` y otra para el puerto `1502` con descripción `Autonomous`. Luego confirma con `Add Ingress Rules`.

![img17](./img/img17.png)

**Step 18**

Verifica que ambas reglas queden creadas en la lista.

![img18](./img/img18.png)

---

### Paso 3: Preparar la llave pública SSH

**Step 19**

En tu equipo local, verifica que en la carpeta `oci` existan `config`, `key.pem` y `key_public.pem`.

![img19](./img/img19.png)

**Step 20**

Abre `key_public.pem` y confirma que contiene la clave pública. vamos a modificarla para que quede en una sola linea

```sql
ssh-rsa <key_public.pem in line>
```


![img20](./img/img20.png)

**Step 21**

Guarda una copia de esa clave pública con el nombre `key_agent_factoru_public.pem`. Ese archivo es el que usarás al crear la VM del stack.

![img21](./img/img21.png)

---

### Paso 4: Configurar el stack y aprovisionar la VM

**Step 22**

Vuelve al asistente de `Create stack` y recarga la pantalla si todavía no aparecen la red o la subred recién creadas.

![img22](./img/img22.png)

**Step 23**

Configura las variables del stack con valores como estos:

- `Region: us-chicago-1`
- `VM Compartment: ora26ai`
- `Subnet Compartment: ora26ai`
- `Existing VCN: vcn_name`
- `Existing Subnet: public subnet-vcn_name (Regional)`
- `Public or Private subnet: public`
- `Agent Factory server display name: AgentFactoryVM`
- `Agent Factory server shape: VM.Standard.E5.Flex`
- `OCPUs: 8`
- `Memory (GB): 64`
- `Public ssh key file: files/oci/key_agent_factoru_public.pem`

Luego haz clic en `Next`.

![img23](./img/img23.png)

**Step 24**

En la pantalla de revisión, deja marcada la opción `Run apply` y pulsa `Create`.

![img24](./img/img24.png)

**Step 25**

Se abrirá el `Apply job` en `Resource Manager`; confirma que el estado inicial sea `Accepted`.

![img25](./img/img25.png)

**Step 26**

Espera a que el job cambie a `Succeeded`.

![img26](./img/img26.png)

**Step 27**

Revisa los `Outputs` del job y copia la URL `Agent_Factory_URL`; esa URL te llevará a la instalación web de la solución.

![img27](./img/img27.png)

---

### Paso 5: Configurar la base de datos para Agent Factory

**Step 28**

Si al abrir la URL del agente aparece una advertencia de certificado en el navegador o antivirus, revisa el detalle y continúa solo si tu entorno lo permite.

![img28](./img/img28.png)

**Step 29**

En la pantalla inicial de `AI Database Private Agent Factory`, registra el usuario administrador de la aplicación y continúa con `Next`.

![img29](./img/img29.png)

**Step 30**

En otra pestaña de OCI, busca `Autonomous AI Database`.

![img30](./img/img30.png)

**Step 31**

Abre la base `ora26ai_name` dentro del compartment del laboratorio.

![img31](./img/img31.png)

**Step 32**

Dentro de la ficha de la base, entra en `Database connection`.

![img32](./img/img32.png)

**Step 33**

Descarga el `wallet` desde `Download wallet`.

![img33](./img/img33.png)

**Step 34**

Define una contraseña para el wallet y descárgalo. En este repositorio tienes un archivo de referencia en `files/Wallet_ora26ainame.zip`.

![img34](./img/img34.png)

**Step 35**

Desde `Database actions`, abre `SQL`.

![img35](./img/img35.png)

**Step 36**

Ejecuta el script de `files/oracle26ai.txt`, pero úsalo solo como plantilla y reemplaza la contraseña por una segura antes de correrlo. El objetivo es crear el usuario técnico `app_user` que Agent Factory usará durante la instalación.

Ejemplo de script sanitizado:

```sql
--1. Create upser for Agent Factory

CREATE USER app_user IDENTIFIED BY <pass> DEFAULT TABLESPACE USERS QUOTA unlimited ON USERS;
 GRANT CONNECT, RESOURCE, CREATE TABLE, CREATE SYNONYM, CREATE DATABASE LINK, CREATE ANY INDEX, INSERT ANY TABLE, CREATE SEQUENCE, CREATE TRIGGER, CREATE USER, DROP USER TO app_user;
 GRANT CREATE SESSION TO app_user WITH ADMIN OPTION;
 GRANT READ, WRITE ON DIRECTORY DATA_PUMP_DIR TO app_user;
 GRANT SELECT ON SYS.V_$PARAMETER TO app_user;
/
```

Confirma en `Script Output` que el usuario y los grants se hayan creado correctamente.

![img36](./img/img36.png)

**Step 37**

Vuelve al asistente de instalación de Agent Factory. En `Database configuration`, elige `Wallet`, carga `files/Wallet_ora26ainame.zip`, usa `Username: app_user`, ingresa la contraseña del usuario y prueba la conexión con `Test connection`.

![img37](./img/img37.png)

**Step 38**

Cuando aparezca `Database connection successful`, continúa con `Next`.

![img38](./img/img38.png)

**Step 39**

En la fase `Installation`, haz clic en `Install`.

![img39](./img/img39.png)

**Step 40**

Espera a que el log muestre que la migración, el registro del usuario y la creación de componentes terminaron correctamente; luego pulsa `Next`.

![img40](./img/img40.png)

---

### Paso 6: Configurar OCI GenAI como LLM

**Step 41**

En OCI, abre el compartment del laboratorio y copia su `OCID`. Lo necesitarás en la configuración del modelo.

![img41](./img/img41.png)

**Step 42**

En `LLM Configuration`, selecciona `OCI GenAI` y completa la configuración del modelo generativo usando tus propios valores:

- `Configuration name: llm_model_entry`
- `Model ID: google.gemini-2.5-pro`
- `Endpoint: https://inference.generativeai.us-chicago-1.oci.oraclecloud.com`
- `Compartment ID: <tu-compartment-ocid>`
- `User: <tu-user-ocid>`
- `Finger Print: <tu-fingerprint>`
- `Tenancy: <tu-tenancy-ocid>`
- `Region: us-chicago-1`
- `Key File: files/oci/key.pem`

Usa `files/oci/config` como referencia para extraer `user`, `fingerprint`, `tenancy`, `region` y la ruta de la llave, pero no copies valores sensibles sin validarlos.

![img42](./img/img42.png)

**Step 43**

Prueba la conexión del modelo generativo con `Test connection`, guarda la configuración con `Save configuration` y luego revisa la sección `Embedding model`. En las capturas se usa el embedding local `multilingual-e5-base`; también se valida con `Test connection`.

![img43](./img/img43.png)

**Step 44**

Guarda ambas configuraciones y finaliza con `Finish Installation`.

![img44](./img/img44.png)

---

### Paso 7: Iniciar sesión y registrar la base como data source

**Step 45**

Cuando la instalación redirija a la pantalla de acceso, introduce el correo y la contraseña del usuario administrador.

![img45](./img/img45.png)

**Step 46**

Haz clic en `Sign In`.

![img46](./img/img46.png)

**Step 47**

Ya dentro del estudio, entra a `Data sources` desde el menú lateral.

![img47](./img/img47.png)

**Step 48**

Pulsa `Add data source`.

![img48](./img/img48.png)

**Step 49**

Registra una nueva fuente de datos de tipo `Database` con valores como estos:

- `Source name: ora26ainame`
- `Description: aidp`
- `Connection type: Wallet`
- `Wallet: files/Wallet_ora26ainame.zip`
- `Username: app_aidp`
- `Password: <password del esquema de datos>`

Luego ejecuta `Test connection`.

![img49](./img/img49.png)

**Step 50**

Cuando la conexión sea exitosa, confirma con `Add database source`.

![img50](./img/img50.png)

**Step 51**

Verifica que la base quede registrada como `connected` y luego entra a `Template gallery`.

![img51](./img/img51.png)

---

### Paso 8: Importar la plantilla y ajustar el flujo conversacional

**Step 52**

En `Template gallery`, importa el flujo `Intent-Based SQL Dispatcher`.

![img52](./img/img52.png)

**Step 53**

Se abrirá el `Agent Builder` con el flujo importado. Revisa la estructura general antes de editarlo.

![img53](./img/img53.png)

**Step 54**

Ajusta el flujo para que use:

- El LLM `llm_model_entry` en los nodos `LLM`
- La base `ora26ainame` en el nodo `SQL query`

![img54](./img/img54.png)

**Step 55**

Abre el nodo `Prompt` que alimenta la consulta base.

![img55](./img/img55.png)

**Step 56**

Reemplaza la plantilla SQL por una consulta a la tabla del caso de uso. En las capturas se utiliza:

```sql
select * from app_aidp.interactions_analysis
```

Guarda con `Finish editing`.

![img56](./img/img56.png)

**Step 57**

Guarda el flujo y abre `Playground` para probarlo.

![img57](./img/img57.png)

**Step 58**

Haz preguntas en lenguaje natural sobre los datos y valida la respuesta. En la captura se prueban preguntas como:

- `¿Cuál fue el usuario con mejor puntuación y cuál fue su comentario?`
- `¿Cuál fue el usuario con mayor toxicidad y cuál fue su comentario?`

![img58](./img/img58.png)

---

## 🧪 Consultas sugeridas para el laboratorio

Puedes probar preguntas como estas en el `Playground`:

- `¿Cuál fue el usuario con mejor puntuación?`
- `¿Qué comentario tuvo la toxicidad más alta?`
- `Muéstrame los comentarios negativos`
- `¿Cuántos registros hay en interactions_analysis?`
- `Resume los usuarios con mayor actividad`

---

## 📁 Archivos de apoyo del laboratorio

| Archivo | Uso en el laboratorio |
|---------|------------------------|
| `files/oracle26ai.txt` | Script base para crear el usuario técnico `app_user` |
| `files/Wallet_ora26ainame.zip` | Wallet usado para la instalación y para el data source |
| `files/oci/config` | Plantilla para extraer `user`, `fingerprint`, `tenancy`, `region` |
| `files/oci/key.pem` | Clave privada para la autenticación con OCI GenAI |
| `files/oci/key_public.pem` | Clave pública original de referencia |
| `files/oci/key_agent_factoru_public.pem` | Clave pública cargada al crear la VM del stack |
| `requirements.txt` | Dependencias base del repositorio |

---

## 📁 Estructura del Repositorio

```text
.
├── files/
│   ├── Wallet_ora26ainame.zip          # Wallet de referencia
│   ├── oracle26ai.txt                  # Script SQL base para Agent Factory
│   └── oci/
│       ├── config                      # Plantilla de configuración OCI
│       ├── key.pem                     # Clave privada OCI
│       ├── key_public.pem              # Clave pública original
│       └── key_agent_factoru_public.pem# Clave pública para la VM
├── img/                                # Capturas del laboratorio (img01.png ... img58.png)
├── requirements.txt                    # Dependencias base
└── README.md                           # Este manual
```

---

## ✅ Resultado esperado

Al finalizar el laboratorio deberías tener:

- Una instancia de `Oracle AI Database Private Agent Factory` desplegada en OCI
- Un `data source` de base de datos registrado y conectado
- Un modelo LLM de `OCI GenAI` configurado y operativo
- Un flujo importado desde `Template gallery` ajustado al caso de uso
- Respuestas conversacionales sobre la tabla `app_aidp.interactions_analysis`

---

## 📚 Referencias

### Oracle AI Database Private Agent Factory

- [Documentación oficial](https://docs.oracle.com/en-us/iaas/agent-factory/index.html)

### Oracle Cloud Marketplace y Resource Manager

- [Marketplace](https://docs.oracle.com/en-us/iaas/Content/Marketplace/home.htm)
- [Resource Manager](https://docs.oracle.com/en-us/iaas/Content/ResourceManager/home.htm)

### Oracle Autonomous Database

- [Autonomous Database](https://docs.oracle.com/en/cloud/paas/autonomous-database/)
- [Database Actions](https://docs.oracle.com/en/cloud/paas/autonomous-database/serverless/adbsb/database-actions.html)

### OCI Generative AI

- [OCI Generative AI](https://docs.oracle.com/en-us/iaas/Content/generative-ai/home.htm)
- [OCI Python SDK](https://github.com/oracle/oci-python-sdk)

---

<!-- MARKDOWN LINKS & IMAGES -->
[issues-shield]: https://img.shields.io/github/issues/othneildrew/Best-README-Template.svg?style=for-the-badge
[issues-url]: https://github.com/jganggini/oracle-ai-database-private-agent-factory/issues
[linkedin-shield]: https://img.shields.io/badge/-LinkedIn-black.svg?style=for-the-badge&logo=linkedin&colorB=555
[linkedin-url]: https://www.linkedin.com/in/jgangini/
