# Pandora — Cronograma del proyecto

Cronograma semanal derivado de `project-timeline.csv`. Cada barra abarca la(s) semana(s) en que se registró ese trabajo.

<span style="color:#6d3bd7">■</span> Backend · <span style="color:#a078ff">■</span> Frontend · <span style="color:#abd600">■</span> Mobile · <span style="color:#a39dab">■</span> Documentación

## Resumen

El trabajo de backend, frontend y mobile unificado en una única línea de desarrollo, con una barra por semana.

```mermaid
%%{init: {
  "theme": "base",
  "themeVariables": {
    "fontSize": "13px",
    "textColor": "#49454e",
    "titleColor": "#22005d",
    "gridColor": "#e6e0e9",
    "sectionBkgColor": "#f7f2fa",
    "altSectionBkgColor": "#ffffff",
    "sectionBkgColor2": "#f7f2fa",
    "taskTextOutsideColor": "#1c1b1f",
    "taskTextDarkColor": "#1c1b1f"
  },
  "gantt": { "barHeight": 24, "barGap": 8, "topPadding": 40, "leftPadding": 130, "gridLineStartPadding": 30, "fontSize": 13, "sectionFontSize": 14, "numberSectionStyles": 2 },
  "themeCSS": ".task0{fill:#6d3bd7;stroke:#4a1fa8} .task1{fill:#cac4d0;stroke:#7a7580} .taskText0{fill:#fff} .taskText1{fill:#1c1b1f} .milestone{fill:#22005d;stroke:#22005d} .sectionTitle{font-weight:600}"
}}%%
gantt
    dateFormat YYYY-MM-DD
    axisFormat %d/%m
    tickInterval 1week
    weekday monday
    todayMarker off

    section Desarrollo
    Inicio backend + web                           :milestone, s0, 2026-08-17, 0d
    Configuración inicial (backend + web)          :s1, 2026-08-17, 7d
    Módulos y entidades base                       :s2, 2026-09-07, 7d
    Funcionalidades base · auth, obras, Q2Q, cartas, batalla :s3, 2026-09-14, 7d
    Moderación, apelaciones, admin, hardening      :s4, 2026-09-21, 7d
    Inicio mobile                                  :milestone, s5, 2026-09-28, 0d
    Correcciones de API, tests web, setup mobile   :s6, 2026-09-28, 7d
    Nuevos casos de uso, sistema de diseño, auth mobile :s7, 2026-10-05, 7d
    Configuración de cuenta, reglas de contraseña, funcionalidades mobile :s8, 2026-10-12, 7d

    section Documentación
    Documentación inicial                          :t1, 2026-09-07, 14d
    Sitio de documentación y docs por funcionalidad :t2, 2026-09-14, 21d
    Casos de uso por archivo, guía de diseño       :t3, 2026-10-05, 14d
```

## Detalle por área

Detalle derivado por commits del proyecto.

```mermaid
%%{init: {
  "theme": "base",
  "themeVariables": {
    "fontSize": "12px",
    "textColor": "#49454e",
    "titleColor": "#22005d",
    "gridColor": "#e6e0e9",
    "sectionBkgColor": "#f7f2fa",
    "altSectionBkgColor": "#ffffff",
    "sectionBkgColor2": "#f7f2fa",
    "taskTextOutsideColor": "#1c1b1f",
    "taskTextDarkColor": "#1c1b1f"
  },
  "gantt": { "barHeight": 20, "barGap": 6, "topPadding": 40, "leftPadding": 100, "gridLineStartPadding": 30, "fontSize": 12, "sectionFontSize": 14, "numberSectionStyles": 4 },
  "themeCSS": ".task0{fill:#6d3bd7;stroke:#4a1fa8} .task1{fill:#a078ff;stroke:#6d3bd7} .task2{fill:#abd600;stroke:#586300} .task3{fill:#cac4d0;stroke:#7a7580} .taskText0,.taskText1{fill:#fff} .taskText2,.taskText3{fill:#1c1b1f} .sectionTitle{font-weight:600}"
}}%%
gantt
    dateFormat YYYY-MM-DD
    axisFormat %d/%m
    tickInterval 1week
    weekday monday
    todayMarker off

    section Backend
    Project init (NestJS scaffold)                       :b1, 2026-08-17, 7d
    Initial modules, entities, artwork upload            :b2, 2026-09-07, 7d
    Rating terminology fix, DB scaffold                  :b3, 2026-09-14, 7d
    Phases 1-2 · auth, roles, users, follows             :b4, 2026-09-14, 7d
    Phase 3 · artworks, Cloudinary, tags, comments, ratings :b5, 2026-09-14, 7d
    Mail, token refresh, cards, scheduler, packages, battle :b6, 2026-09-14, 7d
    Moderation reports, favorites, artwork delete        :b7, 2026-09-14, 7d
    Admin role, appeals, DTO refactor                    :b8, 2026-09-21, 7d
    Hardening · rate limiting, sanitization, tests, logs :b9, 2026-09-21, 7d
    API and qualification fixes, merge to master         :b10, 2026-09-28, 7d
    CU05/07/08/10/11 + avatar upload                     :b11, 2026-10-05, 7d
    Password strength validation                         :b12, 2026-10-12, 7d

    section Frontend
    Project init (Vite + React + TS)                     :f1, 2026-08-17, 7d
    Implementation plan, Phase 0 foundations             :f2, 2026-09-14, 7d
    Phases 1-2 · auth and users                          :f3, 2026-09-14, 7d
    Phase 3 · artworks                                   :f4, 2026-09-14, 7d
    Phases 4-7 · Q2Q, cards, packages, battle            :f5, 2026-09-14, 7d
    Phase 8 · reports, notifications, favorites          :f6, 2026-09-21, 7d
    Admin tools, appeals, integration tests              :f7, 2026-09-21, 7d
    Feature refactor                                     :f8, 2026-09-21, 7d
    Admin additions, artwork screen fixes                :f9, 2026-09-28, 7d
    Phase 9 · Vitest/RTL test suite                      :f10, 2026-09-28, 7d
    Battle mechanics overhaul                            :f11, 2026-10-05, 7d
    Landing hero, About page                             :f12, 2026-10-05, 7d
    Design system applied                                :f13, 2026-10-05, 7d
    Favicon and app icons                                :f14, 2026-10-05, 7d
    Home page redesign                                   :f15, 2026-10-12, 7d
    Account settings (CU07/08/31)                        :f16, 2026-10-12, 7d
    Stricter password validation                         :f17, 2026-10-12, 7d

    section Mobile
    Project init (Kotlin + Compose)                      :m1, 2026-09-28, 7d
    Implementation plan and scope                        :m2, 2026-10-05, 7d
    Phases 1-2 · foundations and auth                    :m3, 2026-10-05, 7d
    Phases 3-4 · artworks, profiles, account             :m4, 2026-10-12, 7d
    Phase 5 · Q2Q, cards, packages, favorites            :m5, 2026-10-12, 7d
    Phase 6 · local autobattler                          :m6, 2026-10-12, 7d
    Phase 7 · notifications, reports, appeals            :m7, 2026-10-12, 7d
    Phase 8 · release hardening, i18n, a11y, CI          :m8, 2026-10-12, 7d
    Device testing and fixes                             :m9, 2026-10-12, 7d

    section Docs
    Documentación inicial                                :d1, 2026-09-07, 7d
    Documentación inicial                                :d2, 2026-09-14, 7d
    Docs site seeded, use cases and requirements         :d3, 2026-09-14, 7d
    Admin/appeals docs, Docsify publish                  :d4, 2026-09-21, 7d
    Class and architecture diagrams, system interfaces   :d5, 2026-09-28, 7d
    Use cases split per CU, folder reorg                 :d6, 2026-10-05, 7d
    Design system guide (DESIGN.md)                      :d7, 2026-10-05, 7d
    Frontend tech-debt consolidated                      :d8, 2026-10-12, 7d
```
