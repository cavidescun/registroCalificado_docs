
### sp_GetAspNetUsers

Procedimiento que obtiene información básica del usuario (PrimerNombre, PrimerApellido, Id) que instanció un caso específico en el sistema BPM. Realiza una consulta anidada para correlacionar el número de caso con el usuario responsable de su creación.

#### Diagrama de flujo

```mermaid
flowchart TD
    A[Inicio: sp_GetAspNetUsers] --> B[Recibe @pNumCaso]
    B --> C[SELECT desde BPM4USCUN.dbo.AspNetUsers]
    C --> D[Subconsulta: SELECT InstanciadoPor desde TM_Caso]
    D --> E[WHERE Numero = @pNumCaso]
    E --> F[Correlación por Id = InstanciadoPor]
    F --> G[Retorna PrimerNombre, PrimerApellido, Id]
    G --> H[Fin]
    
    style A fill:#e1f5fe
    style C fill:#fff3e0
    style G fill:#e8f5e8
    style H fill:#e8f5e8
```
#### Procedimiento almacenado
```sql
CREATE PROCEDURE [dbo].[sp_GetAspNetUsers]
@pNumCaso VARCHAR(250)
AS
BEGIN
SELECT PrimerNombre, PrimerApellido, Id
FROM BPM4USCUN.dbo.AspNetUsers
WHERE Id = (SELECT InstanciadoPor
FROM BPM4UsCun.casos.TM_Caso
WHERE Numero = @pNumCaso);
END;
```
#### Operaciones Principales

- Consulta principal: SELECT desde tabla AspNetUsers para datos del usuario
- Subconsulta correlacionada: Obtiene InstanciadoPor desde TM_Caso por número específico
- Filtrado por caso: WHERE con número de caso como parámetro de entrada
- Correlación usuarios: JOIN implícito por Id = InstanciadoPor
- Retorno información: Campos básicos del usuario que creó el caso

#### Tablas afectadas

##### Consultadas:

- BPM4USCUN.dbo.AspNetUsers: Tabla de usuarios del sistema de identidad ASP.NET
- BPM4UsCun.casos.TM_Caso: Tabla de casos del sistema BPM con información de instanciación

#### Procedimientos Almacenados Anidados