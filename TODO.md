# TODO — Backlog

Backlog vivo del repo. Marca las casillas al completar cada punto.

- [Contenido pendiente](#contenido-pendiente)
- [Deuda de formato](#deuda-de-formato)
- [Arreglos puntuales](#arreglos-puntuales)
- [Mejoras de estudio](#mejoras-de-estudio)

---

## Contenido pendiente

Conversión de PDFs de `udemy_notes/` a notas Markdown en `notes/`. Ordenado por peso en el examen.

| Prioridad | Tema | Fuente | Estado |
|---|---|---|---|
| Máxima | 07 — Amazon Bedrock | `07_MLA-C01_bedrock.pdf` | ⬜ Pendiente. |
| Alta | 05 — Model Training, Tuning & Evaluation | `05_MLA-C01_model_training_tunning_evaluation.pdf` | ⬜ Pendiente. |
| Alta | 06 — Generative AI Fundamentals | `06_MLA-C01_gen_ai_fundamentals.pdf` | ⬜ Pendiente. |
| Alta | 08 — ML Operations (MLOps) | `08_MLA-C01_ml_operations.pdf` | ⬜ Pendiente. |
| Media | 12 — Exam Preparation | `12_MLA-C01_exam_preparation.pdf` | ⬜ Pendiente. |
| Media | 09 — Security | `09_MLA-C01_security.pdf` | ⬜ Pendiente. |
| Media | 10 — Management & Governance | `10_MLA-C01_management_governance.pdf` | ⬜ Pendiente. |
| Baja | 11 — ML Best Practices | `11_MLA-C01_ml_best_practice_.pdf` | ⬜ Pendiente. |

- [ ] Crear `notes/00_intro.md` a partir de `00_MLA-C01_intro.pdf`. Hoy ese contenido vive incrustado en el `README.md` en lugar de tener fichero propio.

---

## Deuda de formato

Notas ya escritas que no cumplen las convenciones de `CLAUDE.md`.

- [ ] **`notes/01_data_ingestion_storage.md` — es el fichero más flojo del repo.** No tiene sección `## Why This Topic Matters`, no usa subsecciones `###`, y no incluye ningún callout (`[!TIP]` / `[!IMPORTANT]` / `[!WARNING]`). Es el único que no sigue el patrón de 02–04.
- [ ] **Regla del punto final en `notes/01_data_ingestion_storage.md`.** Unos 87 bullets no terminan en punto.
- [ ] **Regla del punto final en `notes/02_data_transformation.md`.** Unos 122 bullets no terminan en punto.
- [ ] **Regla del punto final en `README.md`.** Unos 7 bullets no terminan en punto.
- [ ] **Añadir intros en prosa por sección en `notes/01_data_ingestion_storage.md`.** Cada sección mayor debe explicar el *por qué* y el modelo mental antes de entrar en bullets o tablas.
- [ ] **Equilibrar el uso de callouts.** `03_aws_managed_services.md` tiene 18 `[!TIP]` pero ningún `[!WARNING]`, y `02_data_transformation.md` solo tiene 1.

> [!NOTE]
> Los ficheros `03_aws_managed_services.md` y `04_sagemaker.md` ya cumplen la regla del punto final. El retrofit solo afecta a 01, 02 y al README.

---

## Arreglos puntuales

- [ ] **Carácter `†` accidental en `notes/04_sagemaker.md`.** Aparece en el heading `## Instance Type Cheat Sheet†` y en su entrada del TOC. Revertirlo antes de commitear.
- [ ] **Título del `README.md` incorrecto.** Dice "AWS Machine Learning **Specialty**" cuando el examen es "Machine Learning Engineer – **Associate**" (MLA-C01).
- [ ] **Frase desactualizada en `README.md` (línea 50).** Dice "All notes are stored as PDFs inside `udemy_notes/`", pero ya hay 4 temas convertidos a Markdown.
- [ ] **Tabla de dominios del `README.md` desalineada con `CLAUDE.md`.** Los nombres de los 11 dominios no cuadran del todo con el mapa por fichero de notas.

---

## Mejoras de estudio

- [ ] **Hoja de trampas transversal de pares distractores.** Reunir en un solo sitio los pares que hoy están dispersos: Glue DataBrew vs. SageMaker Data Wrangler, Ground Truth vs. A2I, EMR vs. Glue para ETL.
- [ ] **Banco de preguntas de práctica propio.** El sitio natural es el tema 12.
