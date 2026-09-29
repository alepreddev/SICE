---

### Archivo 2: `DATABASE.md`

```markdown
# BASE DE DATOS Y MODELADO CON PRISMA (SICE)

## 1. Lineamientos de Base de Datos
* **Motor:** PostgreSQL (alojado en Supabase).
* **Gestión de Cambios:** Exclusivamente mediante `prisma migrate dev`. **Prohibido** crear o modificar tablas manualmente en el panel web de Supabase.
* **Llaves Primarias:** UUIDs generados automáticamente.
* **Escala de Calificación:** Rango de `0.00` a `20.00` (evaluación vigesimal estándar en Venezuela / PNF).

---

## 2. Definición del Esquema: `prisma/schema.prisma`

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// ==========================================
// ENUMERACIONES
// ==========================================

enum UserRole {
  ADMIN
  CONTROL_ESTUDIOS
  COORDINADOR
  DOCENTE
  ESTUDIANTE
}

enum StudentStatus {
  NUEVO_INGRESO
  REGULAR
  CONDICIONADO
  RETIRADO
  EGRESADO
}

enum DedicationType {
  TIEMPO_COMPLETO
  MEDIO_TIEMPO
  CONVENCIONAL
}

enum DegreeType {
  PNF
  CARRERA_TRADICIONAL
  POSTGRADO
}

enum SubjectSpecialType {
  REGULAR
  PROYECTO_SOCIOINTEGRADOR
  SERVICIO_COMUNITARIO
  PASANTIAS
}

enum EnrollmentStatus {
  BORRADOR
  CONFIRMADA
  RETIRADA
}

enum EnrollmentDetailStatus {
  CURSANDO
  APROBADA
  REPROBADA
  RETIRADA
}

enum AttemptType {
  REGULAR
  REPARACION
  EQUIVALENCIA
  SUFICIENCIA
}

// ==========================================
// 1. USUARIOS Y PERFILES
// ==========================================

model User {
  id              String        @id @default(uuid()) @db.Uuid
  email           String        @unique @db.VarChar(255)
  passwordHash    String        @map("password_hash") @db.VarChar(255)
  firstName       String        @map("first_name") @db.VarChar(100)
  lastName        String        @map("last_name") @db.VarChar(100)
  cedulaIdentidad String        @unique @map("cedula_identidad") @db.VarChar(20)
  phone           String?       @db.VarChar(30)
  address         String?       @db.Text
  role            UserRole      @default(ESTUDIANTE)
  isActive        Boolean       @default(true) @map("is_active")
  createdAt       DateTime      @default(now()) @map("created_at")
  updatedAt       DateTime      @updatedAt @map("updated_at")

  student         Student?
  professor       Professor?
  auditLogs       AuditLog[]

  @@map("users")
}

model Student {
  id                String         @id @db.Uuid
  expedienteCodigo  String         @unique @map("expediente_codigo") @db.VarChar(50)
  admissionPeriodId String         @map("admission_period_id") @db.Uuid
  status            StudentStatus  @default(NUEVO_INGRESO)
  createdAt         DateTime       @default(now()) @map("created_at")
  updatedAt         DateTime       @updatedAt @map("updated_at")

  user              User           @relation(fields: [id], references: [id], onDelete: Cascade)
  admissionPeriod   AcademicPeriod @relation(fields: [admissionPeriodId], references: [id], onDelete: Restrict)
  enrollments       Enrollment[]
  academicRecords   StudentAcademicRecord[]

  @@map("students")
}

model Professor {
  id             String         @id @db.Uuid
  codigoDocente  String         @unique @map("codigo_docente") @db.VarChar(50)
  dedicationType DedicationType @default(CONVENCIONAL) @map("dedication_type")
  createdAt      DateTime       @default(now()) @map("created_at")
  updatedAt      DateTime       @updatedAt @map("updated_at")

  user           User           @relation(fields: [id], references: [id], onDelete: Cascade)
  sections       Section[]

  @@map("professors")
}

// ==========================================
// 2. ESTRUCTURA CURRICULAR (CARRERAS Y PNF)
// ==========================================

model Career {
  id                 String       @id @default(uuid()) @db.Uuid
  code               String       @unique @db.VarChar(50)
  name               String       @db.VarChar(255)
  degreeType         DegreeType   @default(PNF) @map("degree_type")
  intermediateDegree String?      @map("intermediate_degree") @db.VarChar(255)
  finalDegree        String       @map("final_degree") @db.VarChar(255)
  createdAt          DateTime     @default(now()) @map("created_at")
  updatedAt          DateTime     @updatedAt @map("updated_at")

  curriculums        Curriculum[]

  @@map("careers")
}

model Curriculum {
  id                   String     @id @default(uuid()) @db.Uuid
  careerId             String     @map("career_id") @db.Uuid
  code                 String     @db.VarChar(50)
  isActive             Boolean    @default(true) @map("is_active")
  totalCreditsRequired Int        @map("total_credits_required")
  createdAt            DateTime   @default(now()) @map("created_at")
  updatedAt            DateTime   @updatedAt @map("updated_at")

  career               Career     @relation(fields: [careerId], references: [id], onDelete: Restrict)
  subjects             Subject[]

  @@unique([careerId, code])
  @@map("curriculums")
}

model Subject {
  id              String             @id @default(uuid()) @db.Uuid
  curriculumId    String             @map("curriculum_id") @db.Uuid
  code            String             @db.VarChar(50)
  name            String             @db.VarChar(255)
  level           Int                // Trayecto / Semestre
  term            Int?               // Tramo (PNF)
  credits         Int
  hoursTheory     Int                @default(0) @map("hours_theory")
  hoursPractice   Int                @default(0) @map("hours_practice")
  hoursLab        Int                @default(0) @map("hours_lab")
  isSpecialType   SubjectSpecialType @default(REGULAR) @map("is_special_type")
  createdAt       DateTime           @default(now()) @map("created_at")
  updatedAt       DateTime           @updatedAt @map("updated_at")

  curriculum      Curriculum         @relation(fields: [curriculumId], references: [id], onDelete: Restrict)
  sections        Section[]
  academicRecords StudentAcademicRecord[]
  prerequisites   Prerequisite[]     @relation("SubjectPrerequisites")
  prerequisiteOf  Prerequisite[]     @relation("IsPrerequisiteOf")

  @@unique([curriculumId, code])
  @@map("subjects")
}

model Prerequisite {
  id                    String   @id @default(uuid()) @db.Uuid
  subjectId             String   @map("subject_id") @db.Uuid
  prerequisiteSubjectId String?  @map("prerequisite_subject_id") @db.Uuid
  requiredCredits       Int      @default(0) @map("required_credits")

  subject               Subject  @relation("SubjectPrerequisites", fields: [subjectId], references: [id], onDelete: Cascade)
  prerequisiteSubject   Subject? @relation("IsPrerequisiteOf", fields: [prerequisiteSubjectId], references: [id], onDelete: Cascade)

  @@unique([subjectId, prerequisiteSubjectId])
  @@map("prerequisites")
}

// ==========================================
// 3. PLANIFICACIÓN Y SECCIONES
// ==========================================

model AcademicPeriod {
  id                 String         @id @default(uuid()) @db.Uuid
  code               String         @unique @db.VarChar(50)
  startDate          DateTime       @map("start_date") @db.Date
  endDate            DateTime       @map("end_date") @db.Date
  enrollmentStart    DateTime       @map("enrollment_start") @db.Date
  enrollmentEnd      DateTime       @map("enrollment_end") @db.Date
  gradeEntryDeadline DateTime       @map("grade_entry_deadline") @db.Date
  isCurrent          Boolean        @default(false) @map("is_current")
  createdAt          DateTime       @default(now()) @map("created_at")
  updatedAt          DateTime       @updatedAt @map("updated_at")

  students           Student[]
  sections           Section[]
  enrollments        Enrollment[]
  academicRecords    StudentAcademicRecord[]

  @@map("academic_periods")
}

model Classroom {
  id        String            @id @default(uuid()) @db.Uuid
  building  String            @db.VarChar(100)
  code      String            @db.VarChar(50)
  capacity  Int
  createdAt DateTime          @default(now()) @map("created_at")
  updatedAt DateTime          @updatedAt @map("updated_at")

  schedules SectionSchedule[]

  @@unique([building, code])
  @@map("classrooms")
}

model Section {
  id               String            @id @default(uuid()) @db.Uuid
  academicPeriodId String            @map("academic_period_id") @db.Uuid
  subjectId        String            @map("subject_id") @db.Uuid
  professorId      String?           @map("professor_id") @db.Uuid
  code             String            @db.VarChar(50)
  maxCapacity      Int               @map("max_capacity")
  isClosed         Boolean           @default(false) @map("is_closed")
  createdAt        DateTime          @default(now()) @map("created_at")
  updatedAt        DateTime          @updatedAt @map("updated_at")

  academicPeriod   AcademicPeriod    @relation(fields: [academicPeriodId], references: [id], onDelete: Restrict)
  subject          Subject           @relation(fields: [subjectId], references: [id], onDelete: Restrict)
  professor        Professor?        @relation(fields: [professorId], references: [id], onDelete: SetNull)
  schedules        SectionSchedule[]
  enrollmentDetails EnrollmentDetail[]
  evaluationPlans  EvaluationPlan[]

  @@unique([academicPeriodId, subjectId, code])
  @@map("sections")
}

model SectionSchedule {
  id          String    @id @default(uuid()) @db.Uuid
  sectionId   String    @map("section_id") @db.Uuid
  classroomId String    @map("classroom_id") @db.Uuid
  dayOfWeek   Int       @map("day_of_week") @db.SmallInt // 1=Lunes, 7=Domingo
  startTime   DateTime  @map("start_time") @db.Time
  endTime     DateTime  @map("end_time") @db.Time

  section     Section   @relation(fields: [sectionId], references: [id], onDelete: Cascade)
  classroom   Classroom @relation(fields: [classroomId], references: [id], onDelete: Restrict)

  @@map("section_schedules")
}

// ==========================================
// 4. INSCRIPCIONES, NOTAS Y HISTORIAL
// ==========================================

model Enrollment {
  id               String             @id @default(uuid()) @db.Uuid
  studentId        String             @map("student_id") @db.Uuid
  academicPeriodId String             @map("academic_period_id") @db.Uuid
  status           EnrollmentStatus   @default(CONFIRMADA)
  enrolledAt       DateTime           @default(now()) @map("enrolled_at")

  student          Student            @relation(fields: [studentId], references: [id], onDelete: Restrict)
  academicPeriod   AcademicPeriod     @relation(fields: [academicPeriodId], references: [id], onDelete: Restrict)
  details          EnrollmentDetail[]

  @@unique([studentId, academicPeriodId])
  @@map("enrollments")
}

model EnrollmentDetail {
  id           String                 @id @default(uuid()) @db.Uuid
  enrollmentId String                 @map("enrollment_id") @db.Uuid
  sectionId    String                 @map("section_id") @db.Uuid
  status       EnrollmentDetailStatus @default(CURSANDO)
  finalGrade   Decimal?               @map("final_grade") @db.Decimal(4, 2)
  createdAt    DateTime               @default(now()) @map("created_at")
  updatedAt    DateTime               @updatedAt @map("updated_at")

  enrollment   Enrollment             @relation(fields: [enrollmentId], references: [id], onDelete: Cascade)
  section      Section                @relation(fields: [sectionId], references: [id], onDelete: Restrict)
  grades       GradeDetail[]

  @@unique([enrollmentId, sectionId])
  @@map("enrollment_details")
}

model EvaluationPlan {
  id         String        @id @default(uuid()) @db.Uuid
  sectionId  String        @map("section_id") @db.Uuid
  name       String        @db.VarChar(150)
  percentage Decimal       @db.Decimal(5, 2)
  dueDate    DateTime?     @map("due_date") @db.Date
  createdAt  DateTime      @default(now()) @map("created_at")

  section    Section       @relation(fields: [sectionId], references: [id], onDelete: Cascade)
  grades     GradeDetail[]

  @@map("evaluation_plans")
}

model GradeDetail {
  id                 String           @id @default(uuid()) @db.Uuid
  enrollmentDetailId String           @map("enrollment_detail_id") @db.Uuid
  evaluationPlanId   String           @map("evaluation_plan_id") @db.Uuid
  score              Decimal          @db.Decimal(4, 2)
  createdAt          DateTime         @default(now()) @map("created_at")
  updatedAt          DateTime         @updatedAt @map("updated_at")

  enrollmentDetail   EnrollmentDetail @relation(fields: [enrollmentDetailId], references: [id], onDelete: Cascade)
  evaluationPlan     EvaluationPlan   @relation(fields: [evaluationPlanId], references: [id], onDelete: Restrict)

  @@unique([enrollmentDetailId, evaluationPlanId])
  @@map("grade_details")
}

model StudentAcademicRecord {
  id               String         @id @default(uuid()) @db.Uuid
  studentId        String         @map("student_id") @db.Uuid
  subjectId        String         @map("subject_id") @db.Uuid
  academicPeriodId String         @map("academic_period_id") @db.Uuid
  attemptType      AttemptType    @default(REGULAR) @map("attempt_type")
  definitiveGrade  Decimal        @map("definitive_grade") @db.Decimal(4, 2)
  isApproved       Boolean        @map("is_approved")
  lockedAt         DateTime       @default(now()) @map("locked_at")

  student          Student        @relation(fields: [studentId], references: [id], onDelete: Restrict)
  subject          Subject        @relation(fields: [subjectId], references: [id], onDelete: Restrict)
  academicPeriod   AcademicPeriod @relation(fields: [academicPeriodId], references: [id], onDelete: Restrict)

  @@unique([studentId, subjectId, academicPeriodId, attemptType])
  @@map("student_academic_records")
}

// ==========================================
// 5. AUDITORÍA
// ==========================================

model AuditLog {
  id         String   @id @default(uuid()) @db.Uuid
  userId     String?  @map("user_id") @db.Uuid
  action     String   @db.VarChar(100)
  entityName String   @map("entity_name") @db.VarChar(100)
  entityId   String   @map("entity_id") @db.Uuid
  oldValues  Json?    @map("old_values")
  newValues  Json?    @map("new_values")
  ipAddress  String?  @map("ip_address") @db.VarChar(50)
  createdAt  DateTime @default(now()) @map("created_at")

  user       User?    @relation(fields: [userId], references: [id], onDelete: SetNull)

  @@map("audit_logs")
}
