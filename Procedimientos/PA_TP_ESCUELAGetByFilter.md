### PA_TP_ESCUELAGetByFilter

Procedimiento CRUD para consultar escuelas con filtros dinámicos y paginación. Permite filtrar por múltiples campos opcionales y maneja cálculos de offset para paginación de resultados, aunque la implementación actual no aplica la paginación calculada.

#### Diagrama de flujo

```mermaid
flowchart TD
    A[Inicio: PA_TP_ESCUELAGetByFilter] --> B[Recibe filtros opcionales<br/>y parámetros paginación]
    B --> C[Calcula @offset y @newsize<br/>según @pag y @TamPag]
    C --> D{¿ @pag = 0?}
    
    D -->|Sí| E[offset = @pag<br/>newsize = @TamPag]
    D -->|No| F[offset = @pag*@TamPag+1<br/>newsize = @TamPag-1]
    
    E --> G[Establece @OrdenBy por defecto = 'Id']
    F --> G
    G --> H[SELECT con filtros dinámicos<br/>usando ISNULL]
    H --> I[WHERE Estado = 1]
    I --> J[Retorna resultados filtrados]
    J --> K[Fin]
    
    style A fill:#e1f5fe
    style K fill:#e8f5e8
    style D fill:#fff3e0
    style H fill:#f3e5f5
    style J fill:#e8f5e8
```
#### Procedimiento almacenado

```sql
-- ******************************************
-- |PA_TP_ESCUELAGetByFilter|/_
-- Empresa: TiGlobal SAS
-- Procedimiento: [API].[PA_TP_ESCUELAGetByFilter]
-- Creado Por: mc.diaz
-- Fecha: Aug 12 2024 9:55AM
-- Proyecto: ProyectoGenerado
-- Descripcion: Parte del CRUD Básico, procedimiento de consulta de todos los registros filtrados y paginados
-- ******************************************
Create Procedure [api].[PA_TP_ESCUELAGetByFilter]
@Id Int = NULL,
@Nombre VarChar(160) = NULL,
@Id_director NVarChar(450) = NULL,
@auditoria VarChar(MAX) = NULL,
@TamPag int, @pag int, @OrdenBy nvarchar(255)
AS
Set Nocount ON
BEGIN
DECLARE @offset INT
DECLARE @newsize INT

    	IF(@pag=0)
    	begin
    	   SET @offset = @pag;
    	   SET @newsize = @Tampag
    	end
    	ELSE
    	begin
    		SET @offset = @pag*@Tampag+1;
    		SET @newsize = @Tampag-1
    	end
    set @OrdenBy=isnull(@OrdenBy,'Id')


      Select Id,Nombre,Id_director,auditoria
      From [CUN].[TP_ESCUELA]
      Where isnull(Nombre,'')= isnull(@Nombre,isnull(Nombre,'')) And

isnull(auditoria,'')= isnull(@auditoria,isnull(auditoria,''))
and Estado=1

    END

```
#### Operaciones Principales

- Filtros opcionales: Usa ISNULL para manejar parámetros NULL de forma dinámica
- Cálculo de paginación: Calcula offset y newsize (aunque no se implementan)
- Filtrado condicional: Aplica filtros solo cuando los parámetros tienen valor
- Ordenamiento configurable: Permite especificar campo de ordenamiento

#### Tablas afectadas

- [CUN].[TP_ESCUELA]: Tabla principal de escuelas (solo lectura)

#### Procedimientos Almacenados Anidados