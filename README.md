# Dr. Juan Octaviano González Ambriz

**Médico radiólogo** · Imagenología Diagnóstica y Terapéutica (UNAM) · Morelia,
Michoacán, México
[LinkedIn](https://www.linkedin.com/in/octajuan) · octajuan@outlook.com

Soy radiólogo adscrito en el Hospital de la Mujer (IMSS-Bienestar) y en una
clínica de diagnóstico por imagen en Huandacareo, Michoacán. Además de
interpretar estudios, construyo el software con el que trabajo: registro de
pacientes, visor DICOM y editor de informes, en uso clínico diario.

Me interesa la inteligencia artificial aplicada al trabajo real del radiólogo:
el reporte estructurado, el dictado y el apoyo a la interpretación, medidos por
si ahorran tiempo y errores en la lectura, no por cómo se ven en una demo.

> **In English.** Practicing radiologist (UNAM) who also builds clinical
> software. I wrote and use daily a web platform for patient registration, DICOM
> viewing (Cornerstone3D) and structured radiology reporting. My residency thesis
> applied transfer learning to thyroid nodule biopsy triage. Interested in AI for
> radiology workflow: structured reporting, dictation and reading support.

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
| **Visor local gratuito** · [lucidapacs.com](https://lucidapacs.com) | Cualquier persona | Abre CD, USB o carpetas DICOM en el navegador; las imágenes no salen del equipo. |

Cada superficie ve solo lo que le corresponde: el médico, sus pacientes; el
centro, sus referencias; el paciente, su propio expediente. El control de acceso
se aplica en el servidor en cada consulta, no solo en la interfaz.

#### Visor

Un solo núcleo de visualización sobre Cornerstone3D atiende el archivo clínico,
el visor local y los demos:

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
