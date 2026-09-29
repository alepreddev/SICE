# ARQUITECTURA DEL SISTEMA INTEGRADO DE CONTROL DE ESTUDIOS (SICE)

## 1. Visión General y Patrón de Proyecto
SICE es un sistema universitario fullstack estructurado como un **Monolito Fullstack Integrado en TypeScript** en un único repositorio y con un solo `package.json`.

* **Servidor Unificado**: NestJS actúa como backend API (rutas `/api/*`) y como servidor de archivos estáticos (`@nestjs/serve-static`) para el frontend compilado.
* **Separación de Responsabilidades**: Aunque conviven en la misma raíz, la lógica de servidor y de cliente se mantienen estrictamente aisladas en sus directorios respectivos.

---

## 2. Stack Tecnológico

| Capa | Tecnología | Rol Principal |
| :--- | :--- | :--- |
| **Lenguaje Base** | TypeScript 5.x | Tipado estático de punta a punta |
| **Backend / API** | Node.js + NestJS | Controladores, servicios, transacciones y seguridad RBAC |
| **Frontend / UI** | React 18+ con Vite | Single Page Application (SPA), vistas interactivas |
| **ORM & Migraciones**| Prisma ORM | Modelado relacional declarativo, migraciones y tipos |
| **Base de Datos** | PostgreSQL (Supabase) | Motor relacional transaccional (vía conexión directa/pooler) |
| **Estilos / Componentes** | Tailwind CSS / Shadcn UI | Interfaz limpia, responsiva y accesible |
| **Data Fetching** | TanStack Query (React Query) | Cacheo, mutaciones y sincronización del estado del servidor |

---

## 3. Estructura de Directorios

```text
sice-project/
├── prisma/
│   ├── schema.prisma              # Definición de modelos de base de datos
│   └── migrations/                # Migraciones generadas por Prisma
│
├── src/
│   ├── server/                    # CÓDIGO BACKEND (NestJS)
│   │   ├── common/                # Guards, decorators, interceptors, prisma service
│   │   ├── modules/               # Módulos por dominio
│   │   │   ├── auth/              # Autenticación, JWT, hashing
│   │   │   ├── users/             # Gestión de perfiles, estudiantes y docentes
│   │   │   ├── curriculum/        # Carreras/PNF, mallas, unidades curriculares
│   │   │   ├── planning/          # Periodos, aulas, secciones y horarios
│   │   │   ├── enrollments/       # Motor de inscripción y validación de cupos
│   │   │   ├── grades/            # Planes de evaluación, actas y notas
│   │   │   └── audit/             # Registro de auditoría
│   │   ├── app.module.ts          # Módulo raíz
│   │   └── main.ts                # Bootstrap de NestJS
│   │
│   ├── client/                    # CÓDIGO FRONTEND (React)
│   │   ├── assets/                # Imágenes, logos institucionales, íconos
│   │   ├── components/            # UI compartida (botones, tablas, modales)
│   │   ├── features/              # Componentes agrupados por módulo funcional
│   │   ├── hooks/                 # Custom hooks de React / TanStack Query
│   │   ├── services/              # Clientes HTTP (Axios / Fetch) apuntando a /api
│   │   ├── App.tsx
│   │   └── main.tsx               # Bootstrap de React
│   │
│   └── shared/                    # CONTRATOS COMPARTIDOS (TypeScript Puro)
│       ├── types/                 # Interfaces y tipos de payload comunes
│       ├── enums/                 # Enums compartidos (Roles, Estatus, etc.)
│       └── dtos/                  # Esquemas o interfaces de validación
│
├── .env                           # Variables de entorno (DATABASE_URL, JWT_SECRET)
├── tsconfig.json
├── vite.config.ts
└── package.json
