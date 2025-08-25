### PA_TP_GESTORgetByKey

Procedimiento CRUD para obtener un gestor específico por su ID. Retorna el registro completo incluyendo auditoría, validando que el registro esté activo antes de devolverlo.

#### Diagrama de flujo

```mermaid
flowchart TD
    A[Inicio: PA_TP_GESTORgetByKey] --> B[Recibe Id como parámetro]
    B --> C[SET NOCOUNT ON]
    C --> D[SELECT Id, Id_Escuela, Id_gestor, auditoria FROM CUN.TP_GESTOR]
    D --> E[WHERE Id = Id AND Estado = 1]
    E --> F[Retorna registro específico]
    F --> G[Fin]
    
    style A fill:#e1f5fe
    style G fill:#e8f5e8
    style D fill:#f3e5f5
    style F fill:#e8f5e8
```
#### Procedimiento almacenado
```sql
/*
|PA_TP_GESTORgetByKey|/_
Empresa: TiGlobal SAS
Procedimiento: [API].[PA_TP_GESTORgetByKey]
Creado Por: mc.diaz
Fecha: Aug 12 2024 9:55AM
Proyecto: ProyectoGenerado
Descripcion: Parte del CRUD Básico, procedimiento de obtención de un registro
*/
Create Procedure [api].[PA_TP_GESTORgetByKey] @Id Int
AS
Set Nocount ON
BEGIN
select Id,Id_Escuela,Id_gestor,auditoria
from [CUN].[TP_GESTOR]
where Id = @Id
and Estado=1
END
```
#### Operaciones Principales

- Búsqueda por clave primaria: Localiza registro usando ID específico
- Validación de estado: Solo retorna registros activos (Estado = 1)
- Retorno completo: Incluye todos los campos incluyendo auditoría
- Consulta única: Retorna máximo un registro o ninguno

#### Tablas afectadas

- [CUN].[TP_GESTOR]: Tabla principal de gestores (solo lectura)

#### Procedimientos Almacenados Anidados