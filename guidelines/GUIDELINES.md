---

### Archivo 3: `GUIDELINES.md`

```markdown
# GUIDELINES Y ESTÁNDARES DE DESARROLLO (SICE)

Estas directrices son de obligado cumplimiento para todo desarrollador o agente que trabaje en este repositorio con Antigravity.

---

## 1. Convenciones de Nomenclatura

* **Archivos y Directorios:** `kebab-case` (ejemplo: `student-enrollment.service.ts`, `academic-period-card.tsx`).
* **Clases y Componentes:** `PascalCase` (ejemplo: `CurriculumModule`, `GradesTable`).
* **Variables y Funciones:** `camelCase` (ejemplo: `calculateWeightedAverage`, `findSectionById`).
* **Base de Datos (PostgreSQL):** `snake_case` para nombres de tablas y columnas (manejado vía `@map` en Prisma).
* **Constantes y Enums:** `UPPER_SNAKE_CASE` (ejemplo: `UserRole.CONTROL_ESTUDIOS`).

---

## 2. Reglas del Backend (NestJS)

1. **Controladores Delgados, Servicios Ricos:**  
   Los controladores solo reciben DTOs, validan con `ValidationPipe` y delegan la lógica a los servicios. No debe haber lógica de cálculo o sentencias complejas en los controladores.
2. **Uso de Transacciones en Operaciones Sensibles:**  
   Todo flujo que afecte cupos, inscripciones o actas de notas debe estar envuelto en:
   ```typescript
   await this.prisma.$transaction(async (tx) => {
     // Lógica con tx...
   });
