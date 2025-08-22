### PA_TP_ESCUELADelete

Procedimiento CRUD para eliminación lógica de escuelas con validación de dependencias. Verifica si existen registros relacionados antes de proceder con la eliminación, actualizando el estado a inactivo (0) en lugar de eliminar físicamente el registro.

#### Diagrama de flujo

```mermaid
flowchart TD
    A[Inicio: PA_TP_ESCUELADelete] --> B[Recibe @Id, @Auditoria]
    B --> C[Declara variables y tabla temporal @tDepenActiva]
    C --> D[EXEC ValidarDependencias<br/>para CUN.TP_ESCUELA]
    D --> E[Cuenta dependencias activas]
    E --> F{¿@cantidad >= 1?}
    
    F -->|Sí| G[Obtiene primera dependencia]
    G --> H[Construye mensaje de error]
    H --> I[RAISERROR y RETURN]
    
    F -->|No| J[UPDATE TP_ESCUELA<br/>SET Estado=0]
    J --> K[Actualiza Auditoria]
    
    I --> L[Fin con error]
    K --> M[Fin exitoso]
    
    style A fill:#e1f5fe
    style L fill:#ffebee
    style M fill:#e8f5e8
    style F fill:#fff3e0
    style I fill:#ffebee
    style J fill:#f3e5f5
```

#### Procedimiento almacenado

```sql
-- ********************
--|PA_TP_ESCUELADelete|/_
-- Empresa: TiGlobal SAS
-- Procedimiento: [API].[PA_TP_ESCUELADeleteLog]
-- Creado Por: mc.diaz
-- Fecha: Aug 12 2024 9:55AM
-- Proyecto: ProyectoGenerado
-- Descripcion: Parte del CRUD Básico, procedimiento de eliminación lógica
-- *******************
Create Procedure [api].[PA_TP_ESCUELADelete] @Id Int , @Auditoria VarChar(MAX)
AS
Set Nocount ON
BEGIN
-- DECLARACION DE TABLAS
-----------------------------------------
declare @cantidad int
declare @esquema varchar(200)
declare @tabla varchar(200)
declare @sentencia varchar(1000)
declare @tDepenActiva table(
table_schema varchar(255)
,table_name varchar(255)
,cantidad int
)
-----------------------------------------
insert into @tDepenActiva
exec [dbo].[ValidarDependencias] 'CUN','TP_ESCUELA', @Id
-------
set @cantidad = 0
select @cantidad = count(0) from @tDepenActiva
if @cantidad >= 1
begin
select top 1 @esquema = table_schema, @tabla = table_name
from @tDepenActiva
set @sentencia = 'Hay dependencia en la tabla: '+ @esquema+'.'+ @tabla
raiserror (@sentencia, 16, 1 )
return
end
---------------------------------------
update [CUN].[TP_ESCUELA]
set Estado=0
, Auditoria = @Auditoria
where Id = @Id
and Estado=1
-----------------------------------------
END
```

#### Operaciones Principales

- Validación de dependencias: Ejecuta ValidarDependencias para verificar registros relacionados
- Control de integridad: Previene eliminación si existen dependencias activas
- Eliminación lógica: Actualiza Estado=0 en lugar de DELETE físico
- Manejo de errores: Lanza error descriptivo si hay dependencias

#### Tablas afectadas

- [CUN].[TP_ESCUELA]: Tabla principal de escuelas (actualización)
- @tDepenActiva: Tabla temporal para validar dependencias (escritura/lectura)

#### Procedimientos Almacenados Anidados

- [dbo].[ValidarDependencias]: Procedimiento que verifica dependencias en otras tablas para el esquema 'CUN', tabla 'TP_ESCUELA' y ID específico

