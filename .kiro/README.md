# Configuración de Kiro para MongoDB University

Este directorio contiene la configuración personalizada de Kiro para el proyecto de documentación de MongoDB University.

## Hooks Disponibles

### MongoDB Documentation Enhancer

**Ubicación:** `.kiro/hooks/markdown-documentation-enhancer.json`

**Propósito:** Mejora automática de documentos Markdown con estándares profesionales para notas de estudio de MongoDB University.

#### Cómo Usar

1. Abre cualquier archivo Markdown en el editor
2. Escribe en el chat de Kiro una de estas frases:
   - `mejorar documento`
   - `enhance document`
   - `fix markdown`
3. Kiro aplicará automáticamente todas las mejoras

#### Transformaciones Aplicadas

El hook realiza las siguientes mejoras:

- ✅ **Encabezado estructurado** con título, fecha de actualización y descripción
- ✅ **Índice completo** con enlaces navegables a todas las secciones
- ✅ **Mejores prácticas de Markdown** (títulos jerárquicos, formato consistente)
- ✅ **Conceptos ampliados** (mínimo 500 caracteres por concepto)
- ✅ **Corrección gramatical** y eliminación de emojis
- ✅ **Comentarios en código** explicativos
- ✅ **Referencias y fuentes** citadas al final

#### Ejemplo de Uso

```markdown
Antes:
# notas de mongodb
- concepto 1
- concepto 2

Después:
# Introducción a MongoDB Atlas
**Última actualización:** 16 de septiembre de 2026
Descripción del contenido...

## Índice
- [Conceptos](#conceptos)
...

## Conceptos
### Concepto 1
[500+ caracteres de explicación detallada]
```

## Requisitos

- Kiro IDE instalado
- Los hooks se activan automáticamente al iniciar una sesión de Kiro

## Contribuciones

Si mejoras el hook o agregas nuevos, documenta los cambios aquí.

## Notas

- El hook no modifica el contenido técnico original
- Solo aplica formato y amplía explicaciones
- No agrega imágenes automáticamente
- Mantiene el idioma original del documento (español)
