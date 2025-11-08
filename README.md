```SQL

CREATE DATABASE SistemaAsistenciaDB;

GO
USE SistemaAsistenciaDB;
GO

-- TABLAS MAESTRAS
-- ROLES
IF OBJECT_ID('Roles', 'U') IS NOT NULL DROP TABLE Roles;
CREATE TABLE Roles (
    RoleId INT IDENTITY(1,1) PRIMARY KEY,
    Nombre NVARCHAR(50) NOT NULL UNIQUE,
    Descripcion NVARCHAR(200)
);
GO

-- USUARIOS
IF OBJECT_ID('Usuarios', 'U') IS NOT NULL DROP TABLE Usuarios;
CREATE TABLE Usuarios (
    UserId INT IDENTITY(1,1) PRIMARY KEY,
    Username NVARCHAR(100) NOT NULL UNIQUE,
    PasswordHash VARBINARY(MAX) NOT NULL,
    NombreCompleto NVARCHAR(200) NOT NULL,
    Email NVARCHAR(150),
    RoleId INT NOT NULL,
    FotoPerfil VARBINARY(MAX),
    IsActive BIT NOT NULL DEFAULT 1,
    FOREIGN KEY (RoleId) REFERENCES Roles(RoleId)
);
GO

-- DOCENTES
IF OBJECT_ID('Docentes', 'U') IS NOT NULL DROP TABLE Docentes;
CREATE TABLE Docentes (
    DocenteId INT IDENTITY(1,1) PRIMARY KEY,
    UserId INT NOT NULL UNIQUE,
    CodigoDocente NVARCHAR(50),
    Telefono NVARCHAR(50),
    FOREIGN KEY (UserId) REFERENCES Usuarios(UserId)
);
GO

-- ALUMNOS
IF OBJECT_ID('Alumnos', 'U') IS NOT NULL DROP TABLE Alumnos;
CREATE TABLE Alumnos (
    AlumnoId INT IDENTITY(1,1) PRIMARY KEY,
    UserId INT NOT NULL UNIQUE,
    NombreCompleto NVARCHAR(200) NOT NULL,
    Seccion NVARCHAR(10),
    CicloCodigo NVARCHAR(20),
    Foto VARBINARY(MAX),
    FOREIGN KEY (UserId) REFERENCES Usuarios(UserId)
);
GO

-- CICLOS
IF OBJECT_ID('Ciclos', 'U') IS NOT NULL DROP TABLE Ciclos;
CREATE TABLE Ciclos (
    CicloId INT IDENTITY(1,1) PRIMARY KEY,
    Codigo NVARCHAR(50) NOT NULL,
    Descripcion NVARCHAR(200),
    FechaInicio DATE,
    FechaFin DATE,
    IsActive BIT NOT NULL DEFAULT 1
);
GO

-- CURSOS
IF OBJECT_ID('Cursos', 'U') IS NOT NULL DROP TABLE Cursos;
CREATE TABLE Cursos (
    CursoId INT IDENTITY(1,1) PRIMARY KEY,
    Nombre NVARCHAR(100) NOT NULL,
    Descripcion NVARCHAR(200),
    Creditos INT
);
GO

-- AULAS
IF OBJECT_ID('Aulas', 'U') IS NOT NULL DROP TABLE Aulas;
CREATE TABLE Aulas (
    AulaId INT IDENTITY(1,1) PRIMARY KEY,
    Nombre NVARCHAR(100) NOT NULL,
    Capacidad INT,
    CicloId INT NULL,
    IsActive BIT NOT NULL DEFAULT 1,
    FOREIGN KEY (CicloId) REFERENCES Ciclos(CicloId)
);
GO

-- ASIGNACIONES
IF OBJECT_ID('Asignaciones', 'U') IS NOT NULL DROP TABLE Asignaciones;
CREATE TABLE Asignaciones (
    AsignacionId INT IDENTITY(1,1) PRIMARY KEY,
    CursoId INT NOT NULL,
    DocenteId INT NOT NULL,
    AulaId INT NULL,
    CicloId INT NULL,
    Seccion NVARCHAR(10),
    FOREIGN KEY (CursoId) REFERENCES Cursos(CursoId),
    FOREIGN KEY (DocenteId) REFERENCES Docentes(DocenteId),
    FOREIGN KEY (AulaId) REFERENCES Aulas(AulaId),
    FOREIGN KEY (CicloId) REFERENCES Ciclos(CicloId)
);
GO

-- INSCRIPCIONES (Alumno - Asignacion)
IF OBJECT_ID('Inscripciones', 'U') IS NOT NULL DROP TABLE Inscripciones;
CREATE TABLE Inscripciones (
    InscripcionId INT IDENTITY(1,1) PRIMARY KEY,
    AlumnoId INT NOT NULL,
    AsignacionId INT NOT NULL,
    FechaRegistro DATETIME DEFAULT GETDATE(),
    FOREIGN KEY (AlumnoId) REFERENCES Alumnos(AlumnoId),
    FOREIGN KEY (AsignacionId) REFERENCES Asignaciones(AsignacionId)
);
GO

-- NOTAS
IF OBJECT_ID('Notas', 'U') IS NOT NULL DROP TABLE Notas;
CREATE TABLE Notas (
    NotaId INT IDENTITY(1,1) PRIMARY KEY,
    AlumnoId INT NOT NULL,
    AsignacionId INT NOT NULL,
    PC1 DECIMAL(5,2) DEFAULT 0,
    PC2 DECIMAL(5,2) DEFAULT 0,
    Parcial DECIMAL(5,2) DEFAULT 0,
    Final DECIMAL(5,2) DEFAULT 0,
    Promedio AS ((ISNULL(PC1,0)+ISNULL(PC2,0)+ISNULL(Parcial,0)+ISNULL(Final,0))/4.0) PERSISTED,
    FOREIGN KEY (AlumnoId) REFERENCES Alumnos(AlumnoId),
    FOREIGN KEY (AsignacionId) REFERENCES Asignaciones(AsignacionId)
);
GO


-- DATOS INICIALES

-- ROLES
INSERT INTO Roles (Nombre, Descripcion) VALUES
('ADMIN', 'Administrador del sistema'),
('DOCENTE', 'Usuario docente'),
('ALUMNO', 'Usuario alumno');
GO


-- USUARIOS BASE (Contraseña = "12345" en SHA256)


INSERT INTO Usuarios (Username, PasswordHash, NombreCompleto, Email, RoleId, IsActive)
VALUES 
('admin', 0x5994471ABB01112AFCC18159F6CC74B4F511B99806DA59B3CAF5A9C173CACFC5, 'Administrador del Sistema', 'admin@sistema.com', 1, 1),
('docente1', 0x5994471ABB01112AFCC18159F6CC74B4F511B99806DA59B3CAF5A9C173CACFC5, 'Carlos Ramos', 'docente1@sistema.com', 2, 1),
('alumno1', 0x5994471ABB01112AFCC18159F6CC74B4F511B99806DA59B3CAF5A9C173CACFC5, 'María López', 'alumno1@sistema.com', 3, 1);
GO

-- DOCENTE BASE
INSERT INTO Docentes (UserId, CodigoDocente, Telefono)
SELECT UserId, 'DOC001', '987654321' FROM Usuarios WHERE Username='docente1';
GO

-- ALUMNO BASE
INSERT INTO Alumnos (UserId, NombreCompleto, Seccion, CicloCodigo)
SELECT UserId, 'María López', 'A', '2025-1' FROM Usuarios WHERE Username='alumno1';
GO

-- CICLO BASE
INSERT INTO Ciclos (Codigo, Descripcion, FechaInicio, FechaFin, IsActive)
VALUES ('2025-1', 'Primer ciclo académico 2025', '2025-03-01', '2025-07-31', 1);
GO

-- CURSOS BASE
INSERT INTO Cursos (Nombre, Descripcion, Creditos)
VALUES 
('Programación I', 'Curso introductorio a C#', 4),
('Base de Datos', 'Fundamentos de SQL Server', 3);
GO

-- AULA BASE
INSERT INTO Aulas (Nombre, Capacidad, CicloId)
VALUES ('Aula 101', 30, 1);
GO

-- ASIGNACIÓN BASE
INSERT INTO Asignaciones (CursoId, DocenteId, AulaId, CicloId, Seccion)
VALUES (1, 1, 1, 1, 'A');
GO

-- INSCRIPCIÓN BASE
INSERT INTO Inscripciones (AlumnoId, AsignacionId)
VALUES (1, 1);
GO


SELECT 'Base de datos creada correctamente' AS Resultado;
SELECT * FROM Roles;
SELECT Username, NombreCompleto, Email, RoleId FROM Usuarios;
GO



CREATE TRIGGER trg_AgregarDocente_Automatico
ON Usuarios
AFTER INSERT
AS
BEGIN
    INSERT INTO Docentes (UserId, CodigoDocente, Telefono)
    SELECT 
        i.UserId,
        CONCAT('DOC', RIGHT('000' + CAST(i.UserId AS NVARCHAR(3)), 3)),
        '000000000'
    FROM inserted i
    LEFT JOIN Docentes d ON i.UserId = d.UserId
    WHERE i.RoleId = 2 AND d.UserId IS NULL;
END;
GO
```
