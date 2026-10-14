# Dr. Juan Octaviano González Ambriz

**Médico radiólogo** · Imagenología Diagnóstica y Terapéutica (UNAM) · Morelia,
Michoacán, México
[LinkedIn](https://www.linkedin.com/in/octajuan) · octajuan@outlook.com

Radiólogo con experiencia práctica en desarrollo de software e inteligencia artificial
aplicada a imagen médica. Diseñé, programé y opero en producción una plataforma
radiológica completa: ingreso DICOM, visor web, informe estructurado y entrega
segura de resultados. La uso todos los días para leer e informar estudios.

## Resumen técnico

- **Visor DICOM web sobre WebGL.** Cornerstone3D con MPR ortogonal y oblicuo,
  MIP/MinIP, reconstrucción volumétrica, fusión PET/CT con SUV y mastografía.
  Lectura y escritura de **DICOM SEG** y **DICOM SR**, el punto de integración
  natural para devolver al visor las segmentaciones y hallazgos de un modelo.
- **Inteligencia artificial.** Tesis de especialidad con aprendizaje profundo por
  transferencia para clasificar nódulos tiroideos candidatos a biopsia. El
  informe se guarda como JSON estructurado, no como texto libre, para que pueda
  alimentar plantillas, dictado y LLMs sin reprocesar documentos.
- **LLMs en el desarrollo.** Construyo con agentes de programación (Claude Code,
  Codex) en un flujo con control de versiones, pruebas automatizadas y traspasos
  documentados entre agentes. Sé dónde ayudan y dónde hay que verificar cada
  línea, sobre todo en código clínico.
- **APIs e integración.** Endpoints HTTP propios, tokens firmados con HMAC para
  acceso de pacientes, contratos firmados entre servicios (citas, médicos,
  solicitudes) y control de acceso aplicado en el servidor en cada consulta.
- **Infraestructura serverless.** TypeScript estricto, React 19, Cloudflare
  Workers, D1 (SQLite) con Drizzle ORM y almacenamiento de objetos R2 privado.
  Más de 1,000 casos de prueba automatizados.

## Impacto en la práctica

- **Tiempo de lectura.** El estudio, sus previos y el editor del informe están
  en una sola pantalla. Los estudios anteriores del mismo paciente se recuperan
  y asocian automáticamente, en lugar de buscarlos a mano para comparar.
- **Costos de producción.** El resultado (informe e imágenes) se entrega por un
  enlace privado, sin depender de impresión ni de medios físicos en cada
  estudio. La infraestructura serverless evita servidores propios y licencias
  de un PACS comercial.
- **Flujo de trabajo.** Solicitud, cita, registro, carga DICOM, informe y envío
  ocurren en un solo sistema. Los médicos referentes y los centros de salud
  consultan sus resultados en su propio portal, sin llamar a la clínica.
- **Radiólogos de otros hospitales.** El visor local gratuito abre en el
  navegador el CD o USB que trae el paciente, sin instalar el visor del
  proveedor ni subir las imágenes. Sirve en equipos hospitalarios donde no se
  permite instalar software y en cualquier sistema operativo.

> **In English.** Radiologist (UNAM) with hands-on software engineering and
> applied AI experience. I designed, built and run in production a full
> radiology platform: DICOM ingestion, a WebGL viewer (Cornerstone3D: MPR,
> volume rendering, PET/CT, DICOM SEG/SR), structured JSON reporting and signed
> results delivery on Cloudflare Workers, D1 and R2, with 1,000+ automated tests.
> Residency thesis on deep transfer learning for thyroid nodule biopsy triage.
> I build with LLM coding agents under version control and test discipline.

---

## Proyectos

### 1. Plataforma clínica de Clínica CMDH

**Registro, visor DICOM e informes radiológicos** · TypeScript, React,
Cloudflare Workers, Cornerstone3D, Tiptap · en producción en
[clinicacmdh.com](https://clinicacmdh.com)

Es el sistema con el que opera Clínica CMDH, un centro de diagnóstico en
Huandacareo (ultrasonido, rayos X, mastografía, laboratorio y
electrocardiograma). En mi práctica diaria registro a los pacientes, subo los
estudios DICOM y redacto el informe en el editor integrado, junto al visor, sin
cambiar de sistema. El código vive en un repositorio privado por estar en uso
clínico; puedo dar acceso de lectura temporal para una revisión técnica.

El recorrido de un estudio:

```
 solicitud o cita ──> registro del paciente ──> adquisición y carga DICOM
        ──> interpretación en el visor + informe en el editor integrado
        ──> envío del resultado ──> paciente · médico referente · centro de salud
```

| Superficie | Para quién | Qué hace |
| --- | --- | --- |
| **Sitio público** · [clinicacmdh.com](https://clinicacmdh.com) | Pacientes y médicos | Catálogo de estudios, horarios, ubicación y preparación; entrada a solicitudes, citas y portal médico. |
| **Solicitudes y citas** · [solicitudes](https://solicitudes.clinicacmdh.com) · [citas](https://citas.clinicacmdh.com) | Pacientes y médicos | Solicitud de estudio y cita en línea que llegan directamente al registro de la clínica. |
| **Registro clínico** | Personal de la clínica | Pacientes, visitas y estudios con auditoría; carga de DICOM a almacenamiento privado; expediente longitudinal que recupera estudios previos del mismo paciente. |
| **Visor clínico + editor de informes** | Radiólogo | Visor sobre los originales del archivo, lado a lado con el editor estructurado; versionado, firma y PDF tamaño carta. |
| **Resultado del paciente** | Pacientes | Enlace privado firmado (HMAC, revocable) que se envía por WhatsApp: informe, imágenes y estudios anteriores, sin crear cuenta. |
| **Mi embarazo** | Pacientes obstétricas | Dentro del mismo enlace, una línea del tiempo del embarazo construida con los datos estructurados del informe firmado (FUR, edad gestacional, biometría fetal). |
| **Portal de médicos** · [medicos.clinicacmdh.com](https://medicos.clinicacmdh.com) | Médicos referentes | Cuenta individual: solicitudes digitales, citas y consulta de informes e imágenes de sus pacientes referidos, solo de estudios finalizados. |
| **Portal de centros de salud** | Centros de salud de la región | Cuenta por centro, con el nombre de quien consulta en cada evento de auditoría; solo ve los estudios que ese centro refirió. |
| **Visor LUCIDA gratuito** · [lucidapacs.com](https://lucidapacs.com) | Cualquier persona | Abre CD, USB o carpetas DICOM en el navegador; las imágenes no salen del equipo. |

Cada superficie ve solo lo que le corresponde: el médico, sus pacientes; el
centro, sus referencias; el paciente, su propio expediente. El control de acceso
se aplica en el servidor en cada consulta, no solo en la interfaz.

#### Visor LUCIDA

El visor de la plataforma se llama LUCIDA y su versión local es de uso libre:
**[pruébalo en lucidapacs.com](https://lucidapacs.com)** con un CD, USB o carpeta
DICOM, o con los estudios de demostración. Las imágenes se procesan en tu equipo
y no se suben a ningún servidor.

**Implementación.** Un solo núcleo de visualización (Viewer Core) sobre
Cornerstone3D recibe imágenes de tres fuentes con el mismo contrato, de modo que
el visor gratuito y el clínico son el mismo código, no una versión recortada:

```
 CD / USB / carpeta ──> LocalDiscovery ──> LocalSource ──┐
 manifests estáticos ─────────────────────> DemoSource ──┼──> Viewer Core (Cornerstone3D)
 archivo clínico (R2 privado + D1) ───────> ClinicalSource┘
```

`LocalSource` lee `File`/`Blob` en el navegador, interpreta DICOMDIR (con
respaldo de descubrimiento directo cuando está incompleto), archivos sin
extensión, Encapsulated PDF y DICOM SR, sin base de datos ni persistencia.
`ClinicalSource` sirve los originales desde almacenamiento privado con acceso
por capacidad.

**Funciones:**

- Stack 2D con prefetch, ventana/nivel por modalidad, mediciones y anotaciones.
- MPR ortogonal y oblicuo, proyecciones MIP/MinIP, líneas de referencia y
  reconstrucción volumétrica de TC y RM.
- Fusión PET/CT con cálculo de SUV; mastografía con hanging protocol y soporte
  inicial de tomosíntesis.
- Segmentación con estadísticas y superficie 3D; lectura y escritura de
  **DICOM SEG** y **DICOM SR**, el formato en que un modelo de IA puede devolver
  sus resultados al visor.

#### Informe radiológico

- Editor estructurado sobre Tiptap/ProseMirror; el informe se guarda como JSON
  validado, no como HTML ni como un documento de Word. Esa estructura es la base
  para plantillas, dictado e IA.
- Pegado clínico desde Word o desde un LLM: se limpia el ruido de formato y se
  conservan párrafos, listas, negritas y tablas.
- Corrector ortográfico local con diccionario es-MX y un léxico médico propio
  (12 especialidades + CIE-10) que hereda flexiones: si acepta *hipoecoico*,
  acepta *hipoecoicas*. Corre en el navegador y no envía texto a terceros.
- Versionado, PDF tamaño carta con membrete, paginación, firma y código QR.

#### Decisiones de diseño

- **No relajar la geometría para que algo "se vea".** Si una serie de TC tiene
  espaciado irregular o inclinación del gantry, el MPR se deshabilita y se
  explica por qué. Un MPR con geometría falsa es un error diagnóstico esperando
  a ocurrir.
- **Degradación honesta.** Un archivo incompatible produce un mensaje
  recuperable, nunca una clasificación inventada.
- **El informe es dato.** JSON estructurado como formato canónico; el PDF es una
  salida, no la fuente.
- **Local significa local.** En el visor gratuito ninguna imagen sale del equipo.

| Capa | Tecnología |
| --- | --- |
| Lenguaje | TypeScript (estricto) |
| UI | React 19, Tailwind 4 |
| Imagen | Cornerstone3D 5.8, dicom-image-loader, dcmjs, dicom-parser |
| Informe | Tiptap 3 / ProseMirror, nspell, jsPDF |
| Backend | Cloudflare Workers, D1 (SQLite) + Drizzle ORM, R2 |
| Pruebas | `node --test`: 128 archivos, más de 1,000 casos |

Pendiente: soporte clínico completo de tomosíntesis y cine de ultrasonido, y el
dictado por voz con asistencia de IA en el informe. No es un dispositivo médico
certificado; el diagnóstico es responsabilidad del médico que interpreta. Los
datos de demostración son sintéticos o de datasets públicos desidentificados,
con su atribución y licencia.

### 2. Aprendizaje profundo para nódulos tiroideos

Tesis de especialidad (UNAM, 2025): *Desarrollo de un modelo de aprendizaje
profundo para la discriminación de nódulos tiroideos susceptibles a biopsia
mediante transferencia de aprendizaje.*

### 3. Centro Diagnóstico Huandacareo

Perfil público de la clínica y del trabajo de software que hacemos para ella:
[cdhuandacareo](https://github.com/octajuan/cdhuandacareo).

## Publicaciones

- Ambriz JOG, Macías AP. Case report on diaphragmatic hernia with renal ectopia
  as a cause of low back pain and dyspnea. *Asian J Res Rep Urol.*
  2024;7(1):41–45. [doi:10.9734/ajrru/2024/v7i1117](https://doi.org/10.9734/ajrru/2024/v7i1117)
