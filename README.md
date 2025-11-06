```SQL

    CREATE DATABASE SistemaAsistenciaDB;



USE SistemaAsistenciaDB;
GO

-- Roles
IF OBJECT_ID('dbo.Roles', 'U') IS NULL
CREATE TABLE dbo.Roles (
    RoleId INT IDENTITY(1,1) PRIMARY KEY,
    Nombre NVARCHAR(50) NOT NULL,
    Descripcion NVARCHAR(200) NULL
);
GO

-- Usuarios
IF OBJECT_ID('dbo.Usuarios', 'U') IS NULL
CREATE TABLE dbo.Usuarios (
    UserId INT IDENTITY(1,1) PRIMARY KEY,
    Username NVARCHAR(100) NOT NULL UNIQUE,
    PasswordHash VARBINARY(MAX) NOT NULL,
    NombreCompleto NVARCHAR(200) NOT NULL,
    Email NVARCHAR(150) NULL,
    RoleId INT NOT NULL,
    FotoPerfil VARBINARY(MAX) NULL,
    IsActive BIT NOT NULL DEFAULT(1),
    CreatedAt DATETIME NOT NULL DEFAULT(GETDATE()),
    CONSTRAINT FK_Usuarios_Roles FOREIGN KEY(RoleId) REFERENCES dbo.Roles(RoleId)
);
GO

-- Ciclos
IF OBJECT_ID('dbo.Ciclos', 'U') IS NULL
CREATE TABLE dbo.Ciclos (
    CicloId INT IDENTITY(1,1) PRIMARY KEY,
    Codigo NVARCHAR(20) NOT NULL,
    Descripcion NVARCHAR(200) NULL,
    FechaInicio DATE NULL,
    FechaFin DATE NULL,
    IsActive BIT NOT NULL DEFAULT(1)
);
GO

-- Aulas
IF OBJECT_ID('dbo.Aulas', 'U') IS NULL
CREATE TABLE dbo.Aulas (
    AulaId INT IDENTITY(1,1) PRIMARY KEY,
    Nombre NVARCHAR(100) NOT NULL,
    Capacidad INT NULL,
    CicloId INT NULL,
    CONSTRAINT FK_Aulas_Ciclos FOREIGN KEY(CicloId) REFERENCES dbo.Ciclos(CicloId)
);
GO

-- Docentes
IF OBJECT_ID('dbo.Docentes', 'U') IS NULL
CREATE TABLE dbo.Docentes (
    DocenteId INT IDENTITY(1,1) PRIMARY KEY,
    UserId INT NOT NULL UNIQUE,
    CodigoDocente NVARCHAR(50) UNIQUE NULL,
    Telefono NVARCHAR(50) NULL,
    CONSTRAINT FK_Docentes_Usuarios FOREIGN KEY(UserId) REFERENCES dbo.Usuarios(UserId)
);
GO

-- Alumnos
IF OBJECT_ID('dbo.Alumnos', 'U') IS NULL
CREATE TABLE dbo.Alumnos (
    AlumnoId INT IDENTITY(1,1) PRIMARY KEY,
    UserId INT NOT NULL UNIQUE,
    CodigoAlumno NVARCHAR(50) UNIQUE NOT NULL,
    AulaId INT NULL,
    CicloId INT NULL,
    FechaNacimiento DATE NULL,
    CONSTRAINT FK_Alumnos_Usuarios FOREIGN KEY(UserId) REFERENCES dbo.Usuarios(UserId),
    CONSTRAINT FK_Alumnos_Aulas FOREIGN KEY(AulaId) REFERENCES dbo.Aulas(AulaId),
    CONSTRAINT FK_Alumnos_Ciclos FOREIGN KEY(CicloId) REFERENCES dbo.Ciclos(CicloId)
);
GO

-- Cursos
IF OBJECT_ID('dbo.Cursos', 'U') IS NULL
CREATE TABLE dbo.Cursos (
    CursoId INT IDENTITY(1,1) PRIMARY KEY,
    Nombre NVARCHAR(200) NOT NULL,
    Descripcion NVARCHAR(300) NULL,
    Creditos INT NULL
);
GO

-- Asignaciones
IF OBJECT_ID('dbo.Asignaciones', 'U') IS NULL
CREATE TABLE dbo.Asignaciones (
    AsignacionId INT IDENTITY(1,1) PRIMARY KEY,
    CursoId INT NOT NULL,
    DocenteId INT NOT NULL,
    AulaId INT NULL,
    CicloId INT NULL,
    Seccion NVARCHAR(50) NULL,
    CONSTRAINT FK_Asignaciones_Cursos FOREIGN KEY(CursoId) REFERENCES dbo.Cursos(CursoId),
    CONSTRAINT FK_Asignaciones_Docentes FOREIGN KEY(DocenteId) REFERENCES dbo.Docentes(DocenteId),
    CONSTRAINT FK_Asignaciones_Aulas FOREIGN KEY(AulaId) REFERENCES dbo.Aulas(AulaId),
    CONSTRAINT FK_Asignaciones_Ciclos FOREIGN KEY(CicloId) REFERENCES dbo.Ciclos(CicloId)
);
GO

-- Inscripciones
IF OBJECT_ID('dbo.Inscripciones', 'U') IS NULL
CREATE TABLE dbo.Inscripciones (
    InscripcionId INT IDENTITY(1,1) PRIMARY KEY,
    AlumnoId INT NOT NULL,
    AsignacionId INT NOT NULL,
    FechaMatricula DATETIME NOT NULL DEFAULT(GETDATE()),
    IsActive BIT NOT NULL DEFAULT(1),
    CONSTRAINT FK_Inscripciones_Alumnos FOREIGN KEY(AlumnoId) REFERENCES dbo.Alumnos(AlumnoId),
    CONSTRAINT FK_Inscripciones_Asignaciones FOREIGN KEY(AsignacionId) REFERENCES dbo.Asignaciones(AsignacionId)
);
GO

-- Notas
IF OBJECT_ID('dbo.Notas', 'U') IS NULL
CREATE TABLE dbo.Notas (
    NotaId INT IDENTITY(1,1) PRIMARY KEY,
    AlumnoId INT NOT NULL,
    AsignacionId INT NOT NULL,
    PC1 DECIMAL(5,2) NULL,
    PC2 DECIMAL(5,2) NULL,
    Parcial DECIMAL(5,2) NULL,
    Final DECIMAL(5,2) NULL,
    Promedio DECIMAL(5,2) NULL,
    FechaRegistro DATETIME NOT NULL DEFAULT(GETDATE()),
    CONSTRAINT FK_Notas_Alumnos FOREIGN KEY(AlumnoId) REFERENCES dbo.Alumnos(AlumnoId),
    CONSTRAINT FK_Notas_Asignaciones FOREIGN KEY(AsignacionId) REFERENCES dbo.Asignaciones(AsignacionId)
);
GO

-- Asistencia
IF OBJECT_ID('dbo.Asistencia', 'U') IS NULL
CREATE TABLE dbo.Asistencia (
    AsistenciaId INT IDENTITY(1,1) PRIMARY KEY,
    AlumnoId INT NOT NULL,
    AsignacionId INT NOT NULL,
    Fecha DATE NOT NULL,
    HoraIngreso DATETIME NULL,
    HoraSalida DATETIME NULL,
    Estado NVARCHAR(20) NULL,
    RegistradoPor INT NULL,
    CONSTRAINT FK_Asistencia_Alumnos FOREIGN KEY(AlumnoId) REFERENCES dbo.Alumnos(AlumnoId),
    CONSTRAINT FK_Asistencia_Asignaciones FOREIGN KEY(AsignacionId) REFERENCES dbo.Asignaciones(AsignacionId),
    CONSTRAINT FK_Asistencia_Usuarios FOREIGN KEY(RegistradoPor) REFERENCES dbo.Usuarios(UserId)
);
GO

-- Auditoría (opcional)
IF OBJECT_ID('dbo.Auditoria', 'U') IS NULL
CREATE TABLE dbo.Auditoria (
    LogId INT IDENTITY(1,1) PRIMARY KEY,
    UserId INT NULL,
    Accion NVARCHAR(200) NULL,
    Fecha DATETIME NOT NULL DEFAULT(GETDATE()),
    Detalle NVARCHAR(MAX) NULL,
    CONSTRAINT FK_Auditoria_Usuarios FOREIGN KEY(UserId) REFERENCES dbo.Usuarios(UserId)
);
GO

--  Insertar datos iniciales mínimos

-- Roles
INSERT INTO dbo.Roles (Nombre, Descripcion)
SELECT 'Administrador','Usuario con todos los permisos' 
WHERE NOT EXISTS (SELECT 1 FROM dbo.Roles WHERE Nombre='Administrador');

INSERT INTO dbo.Roles (Nombre, Descripcion)
SELECT 'Docente','Usuario que imparte clases' 
WHERE NOT EXISTS (SELECT 1 FROM dbo.Roles WHERE Nombre='Docente');

INSERT INTO dbo.Roles (Nombre, Descripcion)
SELECT 'Alumno','Usuario estudiante' 
WHERE NOT EXISTS (SELECT 1 FROM dbo.Roles WHERE Nombre='Alumno');
GO

-- Usuarios de prueba
INSERT INTO dbo.Usuarios (Username, PasswordHash, NombreCompleto, Email, RoleId, IsActive)
SELECT 'admin', HASHBYTES('SHA2_256','admin123'), 'Administrador Principal', 'admin@instituto.com', RoleId, 1
FROM dbo.Roles WHERE Nombre='Administrador'
AND NOT EXISTS (SELECT 1 FROM dbo.Usuarios WHERE Username='admin');

INSERT INTO dbo.Usuarios (Username, PasswordHash, NombreCompleto, Email, RoleId, IsActive)
SELECT 'docente1', HASHBYTES('SHA2_256','docente123'), 'Juan Pérez', 'docente1@instituto.com', RoleId, 1
FROM dbo.Roles WHERE Nombre='Docente'
AND NOT EXISTS (SELECT 1 FROM dbo.Usuarios WHERE Username='docente1');

INSERT INTO dbo.Usuarios (Username, PasswordHash, NombreCompleto, Email, RoleId, IsActive)
SELECT 'alumno1', HASHBYTES('SHA2_256','alumno123'), 'María López', 'alumno1@instituto.com', RoleId, 1
FROM dbo.Roles WHERE Nombre='Alumno'
AND NOT EXISTS (SELECT 1 FROM dbo.Usuarios WHERE Username='alumno1');
GO

-- Docentes
INSERT INTO dbo.Docentes (UserId, CodigoDocente, Telefono)
SELECT UserId, 'DOC001', '987654321'
FROM dbo.Usuarios
WHERE Username = 'docente1'
AND NOT EXISTS (SELECT 1 FROM dbo.Docentes WHERE UserId=(SELECT UserId FROM dbo.Usuarios WHERE Username='docente1'));
GO

-- Ciclo y Aula
INSERT INTO dbo.Ciclos (Codigo, Descripcion, FechaInicio, FechaFin)
SELECT '2025A', 'Ciclo 2025 A', '2025-03-01', '2025-07-31'
WHERE NOT EXISTS (SELECT 1 FROM dbo.Ciclos WHERE Codigo='2025A');

INSERT INTO dbo.Aulas (Nombre, Capacidad, CicloId)
SELECT 'Aula 101', 30, (SELECT TOP 1 CicloId FROM dbo.Ciclos WHERE Codigo='2025A')
WHERE NOT EXISTS (SELECT 1 FROM dbo.Aulas WHERE Nombre='Aula 101');
GO

-- Alumno vinculado
INSERT INTO dbo.Alumnos (UserId, CodigoAlumno, AulaId, CicloId, FechaNacimiento)
SELECT 
    (SELECT UserId FROM dbo.Usuarios WHERE Username='alumno1'),
    'ALU001',
    (SELECT TOP 1 AulaId FROM dbo.Aulas WHERE Nombre='Aula 101'),
    (SELECT TOP 1 CicloId FROM dbo.Ciclos WHERE Codigo='2025A'),
    '2005-06-15'
WHERE NOT EXISTS (SELECT 1 FROM dbo.Alumnos WHERE CodigoAlumno='ALU001');
GO

-- Curso y Asignación
INSERT INTO dbo.Cursos (Nombre, Descripcion, Creditos)
SELECT 'Matemáticas', 'Curso de Matemáticas', 4
WHERE NOT EXISTS (SELECT 1 FROM dbo.Cursos WHERE Nombre='Matemáticas');

INSERT INTO dbo.Asignaciones (CursoId, DocenteId, AulaId, CicloId, Seccion)
SELECT 
    (SELECT TOP 1 CursoId FROM dbo.Cursos WHERE Nombre='Matemáticas'),
    (SELECT TOP 1 DocenteId FROM dbo.Docentes WHERE CodigoDocente='DOC001'),
    (SELECT TOP 1 AulaId FROM dbo.Aulas WHERE Nombre='Aula 101'),
    (SELECT TOP 1 CicloId FROM dbo.Ciclos WHERE Codigo='2025A'),
    'A'
WHERE NOT EXISTS (SELECT 1 FROM dbo.Asignaciones WHERE Seccion='A');
GO

-- Inscripción alumno
INSERT INTO dbo.Inscripciones (AlumnoId, AsignacionId)
SELECT 
    (SELECT TOP 1 AlumnoId FROM dbo.Alumnos WHERE CodigoAlumno='ALU001'),
    (SELECT TOP 1 AsignacionId FROM dbo.Asignaciones)
WHERE NOT EXISTS (SELECT 1 FROM dbo.Inscripciones WHERE AlumnoId=(SELECT TOP 1 AlumnoId FROM dbo.Alumnos WHERE CodigoAlumno='ALU001'));
GO
```
