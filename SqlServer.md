# MS SQL Server
```
-- Tables -------------------------------------------------------
-- --------------------------------------------------------------

-- Data ---------------------------------------------------------
-- --------------------------------------------------------------

-- Stored Procedure ---------------------------------------------
-- --------------------------------------------------------------

-- Query --------------------------------------------------------
-- --------------------------------------------------------------

-- --------------------------------------------------------------
-- --------------------------------------------------------------
```

## Crear Usuario
```
SELECT 
	NAME AS LoginName, 
	TYPE_DESC AS AccountType, 
	create_date, 
	modify_date,
	TYPE
FROM sys.server_principals
WHERE TYPE IN ('S', 'U', 'G');
GO
```
```
CREATE LOGIN testing WITH PASSWORD = 'testing', CHECK_POLICY = OFF;
GO
CREATE DATABASE db_testing
GO
USE db_testing
GO
CREATE USER testing FOR LOGIN testing;
GO
EXEC sp_addrolemember 'db_owner', 'testing';
```
```
USE db_testing
```

## Crear un registro y recibir datos al mismo tiempo
```
INSERT INTO GG_Users 
    (Id, Email, GoogleSUB, GoogleJTI, SqlToken)
OUTPUT inserted.SqlToken
VALUES 
    ('123', 'a@a', '456', '789', NEWID())
```

## Modificar un registro y recibir datos al mismo tiempo
```
UPDATE GG_Users SET 
    Email = @Email,
    GoogleSUB = @GoogleSUB,
    GoogleJTI = @GoogleJTI,
    SqlToken = NEWID()
OUTPUT inserted.SqlToken
WHERE Id = @Id
```

## Crear Tablas
```
CREATE TABLE Mae_Config (
    Id INT PRIMARY KEY IDENTITY(1,1),
    ApiKey varchar(256) NOT NULL
)
GO
```
* 1 a N
```
CREATE TABLE Perfiles (
    Id INT PRIMARY KEY IDENTITY(1,1),
    Nombre VARCHAR(50) NOT NULL,
    UNIQUE(Nombre),
)
GO

CREATE TABLE Usuarios (
    Id VARCHAR(256) PRIMARY KEY,
    Email VARCHAR(100) NOT NULL,
    AuthHash VARBINARY(64) NOT NULL,
    AuthSalt VARBINARY(16) NOT NULL,
    SqlSession VARCHAR(256) ,
    Id_Perfil INT NOT NULL
    UNIQUE(Email),
    FOREIGN KEY (Id_Perfil) REFERENCES Perfiles(Id)
)
GO
```
* N a N
```
CREATE TABLE Cazadores (
    Id INT PRIMARY KEY IDENTITY(1,1),
    Nombre VARCHAR(100) NOT NULL,
    Edad INT NOT NULL,
)
GO

CREATE TABLE Nen (
    Id INT PRIMARY KEY IDENTITY(1,1),
    Nombre VARCHAR(100) NOT NULL,
    Descripcion VARCHAR(256) NOT NULL,
)
GO

CREATE TABLE CazadorNen (
    Id_Cazador INT NOT NULL,
    Id_Nen INT NOT NULL,
    PRIMARY KEY (Id_Cazador, Id_Nen),
    FOREIGN KEY (Id_Cazador) REFERENCES Cazadores(Id),
    FOREIGN KEY (Id_Nen) REFERENCES Nen(Id),
)
GO
```

## Crear Procedimiento Almacenado con Try
```
CREATE PROCEDURE Auth_Register
    @Id VARCHAR(256),
    @Email VARCHAR(100),
    @Usuario VARCHAR(100),
    @AuthHash VARCHAR(256),
    @AuthSalt VARCHAR(256)
AS
BEGIN
    SET NOCOUNT ON;

    IF EXISTS (SELECT Id FROM Auth_Usuario WHERE Email = @Email)
      BEGIN
        SELECT 400 AS StatusCode, 0 AS Id, 'El Usuario ya Existe' AS Msge
        RETURN
      END

    BEGIN TRY
        INSERT INTO Auth_Usuario
          (Id, Email, Usuario, AuthHash, AuthSalt, Id_Perfil)
        VALUES
          (@Id, @Email, @Usuario, @AuthHash, @AuthSalt, 2)

        SELECT 201 AS StatusCode, 0 AS Id, 'Usuario Registrado Correctamente' AS Msge 
    END TRY
    BEGIN CATCH
        SELECT 500 AS StatusCode, 0 AS Id, 'Error al Guardado los Datos (Auth_Register)' AS Msge 
    END CATCH
END
GO
```

## Crear Procedimiento Almacenado con Transaccion
```
CREATE PROCEDURE Cazadores_Insert
    @Nombre AS VARCHAR(50),
    @Edad AS INT
AS
BEGIN
    SET NOCOUNT ON

    BEGIN TRANSACTION

    BEGIN TRY
        INSERT INTO Cazadores
          (Nombre, Edad)
        VALUES
            (@Nombre, @Edad)		
				
        COMMIT TRANSACTION

        SELECT 201 AS StatusCode, 'Datos Guardados Correctamente' AS Msge, SCOPE_IDENTITY() AS Id
    END TRY
    BEGIN CATCH
        ROLLBACK TRANSACTION

        SELECT ERROR_STATE() AS StatusCode, ERROR_MESSAGE() AS Msge, 0 AS Id
    END CATCH
END
GO
```
```
CREATE PROCEDURE GG_GuidesUser_Set
    @Id_Guide INT,
    @Id_User VARCHAR(256),
    @IsCheck BIT
AS
BEGIN
    SET NOCOUNT ON;

    -- Validar parámetros de entrada
    IF @Id_Guide IS NULL OR @Id_User IS NULL OR @IsCheck IS NULL
    BEGIN
        SELECT 0 AS IsSucces, 400 AS StatusCode, 'Parámetros Obligatorios' AS Msge;
        RETURN;
    END;

    -- Verificar si el usuario existe
    IF NOT EXISTS (SELECT Id FROM GG_Users WHERE Id = @Id_User)
    BEGIN
        SELECT 0 AS IsSucces, 404 AS StatusCode, 'No se Encontró el Usuario' AS Msge;
        RETURN;
    END;

    BEGIN TRANSACTION;

    BEGIN TRY
        -- Usar MERGE para manejar la inserción o actualización
        MERGE GG_GuidesUser AS target
        USING (SELECT @Id_Guide AS Id_Guide, @Id_User AS Id_User) AS source
        ON target.Id_Guide = source.Id_Guide AND target.Id_User = source.Id_User
        WHEN MATCHED THEN
            UPDATE SET IsCheck = @IsCheck
        WHEN NOT MATCHED THEN
            INSERT (Id_Guide, Id_User, IsCheck)
            VALUES (@Id_Guide, @Id_User, @IsCheck);

        -- Confirmar la transacción
        COMMIT TRANSACTION;

        -- Devolver éxito
        SELECT 1 AS IsSucces, 200 AS StatusCode, 'Ok' AS Msge;
    END TRY
    BEGIN CATCH
        -- Revertir la transacción en caso de error
        ROLLBACK TRANSACTION;

        -- Devolver error
        SELECT 0 AS IsSucces, ERROR_STATE() AS StatusCode, ERROR_MESSAGE() AS Msge;
    END CATCH;
END;
GO
```

## Insertar Data Desactivado y Activado el Identity
```
SET IDENTITY_INSERT Auth_Perfil ON
GO

INSERT INTO Auth_Perfil
    (Id, Nombre)
VALUES
    (1, 'Admin'),
    (2, 'Usuario')

SET IDENTITY_INSERT Auth_Perfil OFF
GO
```

# Metodos Autenticacion
```
DECLARE @Clave NVARCHAR(100) = 'ABC123'

DECLARE @Salt VARBINARY(16)
SET @Salt = CRYPT_GEN_RANDOM(16)

DECLARE @Hash VARBINARY(64)
SET @Hash = HASHBYTES('SHA2_256', @Clave + CAST(@Salt AS NVARCHAR(32)))

DECLARE @HashProvidedPassword VARBINARY(64)
SET @hashProvidedPassword = HASHBYTES('SHA2_256', @Clave + CAST(@Salt AS NVARCHAR(32)))

UPDATE Usuarios SET SessionCode = NEWID() WHERE Email = @Email
```
