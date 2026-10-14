# LUCIDA PACS

> El código fuente vive en un repositorio privado porque la plataforma está en
> uso clínico real. Este repositorio público lo describe. Para una revisión
> técnica del código, escríbeme y doy acceso de lectura temporal.

Visor DICOM web, PACS y editor de informes radiológicos construidos alrededor de
un solo núcleo de visualización. Lo desarrollo y lo uso yo mismo: soy radiólogo
y todos los días interpreto e informo estudios con esta plataforma en una clínica
de diagnóstico por imagen en Michoacán, México.

> **In English.** LUCIDA PACS is a browser-based DICOM viewer, cloud PACS surface
> and structured radiology reporting editor, written by a practicing radiologist
> and used daily in a diagnostic imaging clinic. One Cornerstone3D-based Viewer
> Core serves local media (CD/USB, nothing uploaded), public de-identified demos
> and the clinical archive. Stack: TypeScript, React 19, Cloudflare Workers, D1,
> R2, Cornerstone3D 5.8, dcmjs, Tiptap/ProseMirror.

En producción: [clinicacmdh.com](https://clinicacmdh.com) · Visor gratuito:
[lucidapacs.com](https://lucidapacs.com)

---

## Por qué existe

Buena parte de los estudios que llegan a un radiólogo en México todavía viajan en
CD o USB, cada uno con el visor de su proveedor, que a veces no abre, pide
instalación o no corre en Mac. Del otro lado, los sistemas de informe que he
usado tratan el texto como un documento de Word con membrete, no como datos.

Quería tres cosas que no encontraba juntas:

1. Abrir cualquier medio en el navegador **sin subir nada** a un servidor.
2. Que el visor del archivo clínico y el visor "gratuito" fueran **el mismo**,
   no una versión recortada.
3. Que el informe se guarde como estructura, para que un día pueda alimentar
   dictado, plantillas e IA sin reescribir todo.

## En producción: clinicacmdh.com

La plataforma no es un prototipo. Es el sistema con el que opera
[Clínica CMDH](https://clinicacmdh.com), un centro de diagnóstico en
Huandacareo, Michoacán (ultrasonido, rayos X, mastografía, laboratorio y
electrocardiograma). En mi práctica diaria registro a los pacientes, subo los
estudios DICOM y redacto el informe en el editor integrado, junto al visor, sin
cambiar de sistema.

El recorrido de un estudio, de la solicitud al resultado:

```
 solicitud o cita ──> registro del paciente ──> adquisición y carga DICOM
        ──> interpretación en el visor clínico + informe en el editor integrado
        ──> envío del resultado ──> paciente · médico referente · centro de salud
```

| Superficie | Para quién | Qué hace |
| --- | --- | --- |
| **Sitio público** · [clinicacmdh.com](https://clinicacmdh.com) | Pacientes y médicos | Catálogo de estudios, horarios, ubicación y preparación; entrada a solicitudes, citas y portal médico. |
| **Solicitudes y citas** · [solicitudes](https://solicitudes.clinicacmdh.com) · [citas](https://citas.clinicacmdh.com) | Pacientes y médicos | Solicitud de estudio y cita en línea que llegan directamente al registro de la clínica. |
| **Registro clínico** | Personal de la clínica | Pacientes, visitas y estudios con auditoría; carga de DICOM a almacenamiento privado; expediente longitudinal que recupera estudios previos del mismo paciente. |
| **Visor clínico + editor de informes** | Radiólogo | El Viewer Core sobre los originales del archivo, lado a lado con el editor estructurado; versionado, firma y PDF tamaño carta. |
| **Resultado del paciente** | Pacientes | Enlace privado firmado (HMAC, revocable) que se envía por WhatsApp: informe, imágenes en el visor y estudios anteriores, sin crear cuenta. |
| **Mi embarazo** | Pacientes obstétricas | Dentro del mismo enlace, una línea del tiempo del embarazo construida con los datos estructurados del informe firmado (FUR, edad gestacional, biometría fetal). |
| **Portal de médicos** · [medicos.clinicacmdh.com](https://medicos.clinicacmdh.com) | Médicos referentes | Cuenta individual: solicitudes digitales, citas y consulta de informes e imágenes de sus pacientes referidos, solo de estudios finalizados. |
| **Portal de centros de salud** | Centros de salud de la región | Cuenta por centro, con el nombre de quien consulta en cada evento de auditoría; solo ve los estudios que ese centro refirió. |
| **Visor gratuito** · [lucidapacs.com](https://lucidapacs.com) | Cualquier persona | Abre CD, USB o carpetas DICOM en el navegador; las imágenes no salen del equipo. |

Cada superficie ve solo lo que le corresponde: el médico, sus pacientes; el
centro, sus referencias; el paciente, su propio expediente. El control de acceso
se aplica en el servidor en cada consulta, no solo en la interfaz.

## Qué hace hoy

**Visor (Viewer Core)**

- Stack 2D con prefetch, ventana/nivel con presets por modalidad, mediciones y
  anotaciones.
- MPR axial/coronal/sagital, MPR oblicuo y proyecciones (MIP/MinIP), con líneas
  de referencia.
- Reconstrucción volumétrica de TC y RM.
- Fusión PET/CT local con cálculo de SUV.
- Mastografía con hanging protocol y presentación por proyección; soporte inicial
  de tomosíntesis (DBT).
- Segmentación con estadísticas y superficie 3D; lectura y escritura de
  **DICOM SEG** y **DICOM SR**.
- Lectura de DICOMDIR (con fallback a descubrimiento directo), archivos sin
  extensión, Encapsulated PDF y PDF convencional.

**Fuentes de datos, un solo visor**

```
 CD / USB / carpeta ──> LocalDiscovery ──> LocalSource ──┐
 manifests estáticos ─────────────────────> DemoSource ──┼──> Viewer Core (Cornerstone3D)
 archivo clínico (R2 privado + D1) ───────> ClinicalSource┘
```

`LocalSource` procesa `File`/`Blob` en el dispositivo: sin D1, sin R2, sin
analytics, sin persistencia. Es el modo que cualquiera puede probar en el sitio.

**Informe radiológico**

- Editor estructurado sobre Tiptap/ProseMirror; el contenido se guarda como JSON
  validado, no como HTML del navegador.
- Pegado clínico desde Word o desde un LLM: se limpia el ruido de Office y se
  conservan párrafos, listas, negritas y tablas sin convertir los saltos visuales
  en saltos reales.
- Corrector ortográfico local con diccionario es-MX y un léxico médico propio
  (12 especialidades + CIE-10) que hereda flexiones: si el léxico acepta
  *hipoecoico*, acepta *hipoecoicas*. Corre en un Web Worker y no envía texto a
  terceros.
- Versionado de informes y PDF tamaño carta con membrete, paginación, firma y
  código QR del estudio en cada hoja.

## Decisiones que me importan

- **Local significa local.** En el modo gratuito ninguna imagen ni metadata sale
  del equipo. No hay "upload de respaldo" escondido.
- **No relajar la geometría para que algo "se vea".** Si una serie de TC tiene
  espaciado irregular o inclinación del gantry, el MPR se deshabilita y se
  explica por qué. Un MPR bonito con geometría falsa es un error diagnóstico
  esperando a ocurrir.
- **Degradación honesta.** Un archivo incompatible produce un mensaje
  recuperable, nunca una clasificación inventada.
- **Un solo Viewer Core.** Cualquier mejora de interacción se hace una vez y la
  heredan el archivo clínico, el visor local y los demos.
- **El informe es dato.** JSON estructurado como formato canónico; el PDF es una
  salida, no la fuente.

## Arquitectura

```
 navegador ──HTTPS──> Cloudflare Worker (Vinext/React, App Router)
                          ├── app/        páginas y API
                          ├── lib/        dominio: DICOM, viewer-core, editor, auth
                          ├── D1 (SQLite) metadatos clínicos vía Drizzle ORM
                          └── R2 privado  originales DICOM con acceso por capacidad
```

| Capa | Tecnología |
| --- | --- |
| Lenguaje | TypeScript (estricto) |
| UI | React 19, Tailwind 4 |
| Imagen | Cornerstone3D 5.8.2, dicom-image-loader, dcmjs, dicom-parser |
| Informe | Tiptap 3 / ProseMirror, nspell, jsPDF, qrcode |
| Backend | Cloudflare Workers, D1 + Drizzle ORM, R2 |
| Pruebas | `node --test`: 128 archivos, más de 1,000 casos |

Las decisiones de arquitectura se documentan como ADRs dentro del repositorio.


## Datos de demostración

Los estudios de demostración son sintéticos o provienen de datasets
públicos desidentificados. Las cabeceras se revisaron para confirmar que no
contienen nombre, ID, fechas ni institución del paciente original. Cada dataset
conserva su atribución y licencia; por ejemplo, la angio-TC de cabeza y cuello proviene de Blagosklonova E, *et al.*
Zenodo; 2025. doi:10.5281/zenodo.15697196 (CC BY 4.0).

El repositorio no contiene datos de pacientes reales.

## Estado y límites

Es un proyecto en desarrollo activo y en uso clínico real, hecho por una sola
persona. Lo que todavía no considero completo:

- Tomosíntesis y cine de ultrasonido no tienen soporte clínico completo.
- Faltan datasets demo definitivos para TC multifásica, RM de encéfalo
  multisecuencia y US multiframe.
- El dictado por voz y la asistencia con IA en el informe están planeados, no
  implementados.

No es un dispositivo médico certificado. Cualquier uso diagnóstico es
responsabilidad del médico que interpreta.

## Autor

**Juan Octaviano González Ambriz**, médico radiólogo (Imagenología Diagnóstica y
Terapéutica, UNAM).
[LinkedIn](https://www.linkedin.com/in/octajuan) · octajuan@outlook.com

## Licencia

Todos los derechos reservados.
