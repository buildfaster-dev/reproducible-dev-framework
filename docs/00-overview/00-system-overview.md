---
title: "System Overview — Entiende el mapa"
version: "1.0"
status: "Borrador inicial"
author: "Jyr"
date: "2025-10-12"
description: "Introducción narrativa al Reproducible Dev Framework: un recorrido para construir y mantener entornos declarativos con intención, claridad y autonomía."
framework: "Reproducible Dev Framework"
narrative_system: "Story-Driven Diátaxis"
---

<!--  
Contexto interno del sistema narrativo  
Este documento pertenece a la capa Overview del sistema Story-Driven Diátaxis  
del proyecto Reproducible Dev Framework.  
-->

# System Overview — Entiende el mapa

> Este documento te orienta dentro del Reproducible Dev Framework.  
> Aquí entenderás cómo leer, recorrer y aplicar las capas que lo componen.

---

**Tabla de contenidos**

1. [Introducción](#1-introducción)  
2. [Propósito del Reproducible Dev Framework](#2-propósito-del-reproducible-dev-framework)  
3. [Estructura general del repositorio](#3-estructura-general-del-repositorio)  
4. [Capas narrativas](#4-capas-narrativas)  
5. [Cómo navegar los documentos](#5-cómo-navegar-los-documentos)  
6. [Convenciones y estilo](#6-convenciones-y-estilo)  
7. [Cierre](#7-cierre)

---

## 1. Introducción

El Reproducible Dev Framework no es un conjunto de tutoriales técnicos.  
Es un recorrido narrativo diseñado para ayudarte a **pensar, construir y mantener entornos declarativos**  
con intención, filosofía y control.

Su objetivo es guiarte paso a paso,  
desde comprender el “por qué” de cada decisión hasta dominar la práctica técnica de reproducirla.  
Cada documento, carpeta y archivo está diseñado para acompañarte en ese proceso.

---

## 2. Propósito del Reproducible Dev Framework

El Reproducible Dev Framework busca **transformar la relación entre desarrollador y entorno**.  
No se trata solo de instalar herramientas, sino de **cultivar autonomía técnica y claridad mental**.

Su filosofía central es simple:

> “Nada global. Todo declarativo, reproducible, portable y controlado por ti.”

El propósito final es que cualquier persona pueda reconstruir su entorno  
en cualquier máquina, en cualquier momento, **sin perder coherencia ni intención.**

---

## 3. Estructura general del repositorio

El repositorio se organiza por **capas narrativas**, no por temas aislados.  
Cada carpeta representa una etapa del viaje dentro del Reproducible Dev Framework.

```
docs/
├── 00-overview/
│   └── 00-system-overview.md      → Entiende el mapa completo
├── 00-reset/
│   └── 00-reset-story.md          → Empieza tu camino (limpieza consciente)
├── 01-foundations/
│   └── 01-foundations-understanding-and-philosophy.md → Comprende los principios
├── 02-nix-setup/
│   └── 02-nix-setup-story.md      → Crea tu base declarativa
├── 03-home-manager/
│   └── 03-home-manager-story.md   → Expande tu control personal
├── 04-tools-and-layers/
│   └── ...                        → Zellij, Neovim, WezTerm, ASDF, Git, Just
└── 05-validation/
    └── ...                        → Validaciones cruzadas y consistencia final
```

---

## 4. Capas narrativas

Cada capa representa una fase de autonomía.  
No se trata de dificultad técnica, sino de **profundidad de conciencia técnica**.

| Capa | Nombre | Enfoque narrativo | Resultado esperado |
|------|---------|-------------------|--------------------|
| 0 | Reset y conciencia | Limpiar con intención | Control sobre tu espacio digital |
| 1 | Fundamento (Nix + Direnv) | Construir la base declarativa | Entorno reproducible |
| 2 | Espacio propio (Zellij, Neovim, WezTerm) | Crear un entorno personalizado | Flujo de trabajo controlado |
| 3 | Identidad técnica (Git + ASDF) | Versionar tu práctica | Control de versiones y coherencia |
| 4 | Autonomía declarativa (Just) | Automatizar sin perder intención | Ritmo personal y eficiencia |
| 5 | Validación y reflejo | Comprobar, aprender, ajustar | Entorno consistente y documentado |

Cada capa cuenta una historia.  
Tu progreso no es lineal ni impuesto: puedes detenerte, retroceder o experimentar.  
Lo importante es mantener la intención.

---

## 5. Cómo navegar los documentos

- **Lee primero.** Antes de ejecutar comandos, comprende el contexto de cada documento.  
- **Sigue el orden de numeración.** La estructura está diseñada para ser reproducible y progresiva.  
- **Observa las transiciones.** Cada documento cierra con una invitación hacia el siguiente.  
- **No copies ciegamente.** Entiende el porqué antes del cómo.  

Cada documento dentro del Reproducible Dev Framework incluye:
1. **Metadatos YAML** → versión, autor, descripción.  
2. **Comentario interno** → contexto dentro del sistema.  
3. **Tabla de contenidos manual** → navegación estructurada.  
4. **Secciones numeradas** → arquitectura lógica y narrativa.  
5. **Cierre narrativo** → conecta con el siguiente paso del viaje.

---

## 6. Convenciones y estilo

- **Idioma:** Español neutro (con tono humano y técnico).  
- **Nombres de archivos y carpetas:** En inglés, para coherencia y compatibilidad.  
- **Contenido:** En español, para comprensión profunda.  
- **Separadores (`---`):** Solo entre grandes secciones.  
- **Citas (`>`):** Usadas para transmitir intención o reflexión, no para decorar.  
- **Numeración:** Cada documento empieza con dos dígitos (`00`, `01`, etc.) para mantener el orden reproducible.  
- **Estructura:** Basada en el framework *Story-Driven Diátaxis* (aprendizaje narrativo progresivo).

---

## 7. Cierre

Has comprendido el mapa.  
Ahora sabes cómo está estructurado el Reproducible Dev Framework y qué representa cada documento.  

> El siguiente paso es limpiar el terreno.  
> Prepara tu entorno y tu mente para comenzar con intención: **Reset Story** te guiará en ese proceso.
