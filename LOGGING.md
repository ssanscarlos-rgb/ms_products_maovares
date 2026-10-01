# Sistema de Logging con Correlation ID - MS Products

## Resumen

Se ha implementado un sistema completo de logging estructurado con correlation ID para el microservicio de productos. Este sistema permite rastrear todas las transacciones a través de todos los componentes y identificar fácilmente dónde ocurren los fallos.

## Características Implementadas

### 1. Correlation ID Filter
- **Archivo**: `CorrelationIdFilter.java`
- **Función**: Genera y propaga un correlation ID único para cada request
- **Características**:
  - Genera UUID único si no se proporciona correlation ID en el header
  - Agrega el correlation ID al MDC para logging automático
  - Incluye el correlation ID en la respuesta HTTP
  - Logs de inicio y fin de request con duración
  - Captura información del cliente (IP, User-Agent)

### 2. Logging Estructurado JSON
- **Archivo**: `logback-spring.xml`
- **Formato**: JSON estructurado para fácil parsing en sistemas de monitoreo
- **Appenders**:
  - Console: Logs en formato JSON a consola
  - File: Logs rotativos a archivo
  - Error File: Logs de error separados con retención extendida
- **Características**:
  - Logs asíncronos para mejor performance
  - Rotación automática por tamaño y tiempo
  - Incluye correlation ID en todos los logs

### 3. Logging por Capas

#### Controllers (`ProductController`)
- Logs de entrada con parámetros de request
- Logs de éxito con información del resultado
- Logs de error con contexto completo
- Tracking de operaciones (GET, POST)

#### Services
- **CreateProductService**: Logs de creación de productos
- **GetProductService**: Logs de búsqueda por ID
- **GetProductsService**: Logs de paginación
- Todos incluyen información contextual y manejo de errores

#### Repository (`ProductMongoAdapter`)
- Logs detallados de operaciones MongoDB
- Tracking de queries con parámetros
- Logs de conversión de documentos
- Información de performance de base de datos

#### Exception Handler (`GlobalExceptionHandler`)
- Logs estructurados de errores
- Diferentes niveles según tipo de error
- Handler genérico para errores inesperados

#### Security Filter (`ClientCertValidationFilter`)
- Logs de validación de certificados
- Tracking de intentos de acceso no autorizados
- Información de IP del cliente

## Configuración

### application.properties
```properties
# Logging levels específicos por componente
logging.level.com.maovares.ms_products.product.infraestructure.web=INFO
logging.level.com.maovares.ms_products.product.application.service=INFO
logging.level.com.maovares.ms_products.product.infraestructure.persistence=INFO
logging.level.com.maovares.ms_products.product.infraestructure.http.filter=INFO

# Logging patterns con correlation ID
logging.pattern.console=%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] [%X{correlationId:-NO_CORRELATION_ID}] %-5level %logger{36} - %msg%n

# Archivos de log
logging.file.name=logs/ms-products.log
logging.file.max-size=100MB
logging.file.max-history=30
```

## Uso del Correlation ID

### En Requests HTTP
```bash
# El correlation ID se puede enviar en el header
curl -H "X-Correlation-ID: 12345-67890" http://localhost:8080/products

# Si no se envía, se genera automáticamente
curl http://localhost:8080/products
```

### En Logs
Todos los logs incluyen el correlation ID en el formato:
```
2024-01-15 10:30:45.123 [http-nio-8080-exec-1] [abc-123-def] INFO  c.m.m.p.web.ProductController - Getting products with pagination
```

### Estructura JSON de Logs
```json
{
  "timestamp": "2024-01-15T10:30:45.123Z",
  "level": "INFO",
  "logger_name": "com.maovares.ms_products.product.web.ProductController",
  "message": "Getting products with pagination - Page: 0, Size: 20",
  "service": "ms-products",
  "thread": "http-nio-8080-exec-1",
  "correlationId": "abc-123-def"
}
```

## Beneficios para Monitoreo en la Nube

### 1. Trazabilidad Completa
- Cada request puede ser rastreada desde el filtro hasta la base de datos
- El correlation ID permite agrupar todos los logs relacionados

### 2. Identificación Rápida de Fallos
- Logs estructurados facilitan el parsing automático
- Información contextual en cada nivel de la aplicación
- Stack traces completos para debugging

### 3. Métricas de Performance
- Duración de requests registrada
- Tiempos de respuesta de base de datos
- Identificación de cuellos de botella

### 4. Seguridad
- Logs de intentos de acceso no autorizados
- Tracking de validación de certificados
- Información de IP y User-Agent

## Archivos de Log Generados

1. **ms-products.log**: Logs generales de la aplicación
2. **ms-products-errors.log**: Solo logs de error (retención 90 días)
3. **ms-products.YYYY-MM-DD.N.log.gz**: Logs rotativos comprimidos

## Ejemplo de Flujo de Logs

Para una request `GET /products/123`:

1. **CorrelationIdFilter**: Genera correlation ID y log de inicio
2. **ClientCertValidationFilter**: Valida certificado y log de éxito
3. **ProductController**: Log de entrada con ID del producto
4. **GetProductService**: Log de búsqueda en servicio
5. **ProductMongoAdapter**: Logs de query MongoDB
6. **ProductController**: Log de respuesta exitosa
7. **CorrelationIdFilter**: Log de fin con duración total

Todos los logs incluyen el mismo correlation ID, permitiendo rastrear la transacción completa.

## Dependencias Agregadas

```xml
<!-- Logback JSON encoder for structured logging -->
<dependency>
    <groupId>net.logstash.logback</groupId>
    <artifactId>logstash-logback-encoder</artifactId>
    <version>9.0</version>
</dependency>
```

## Próximos Pasos Recomendados

1. Configurar un sistema de log aggregation (ELK Stack, Splunk, etc.)
2. Crear dashboards para monitoreo en tiempo real
3. Configurar alertas basadas en patrones de error
4. Implementar métricas de negocio basadas en logs
5. Considerar agregar más contexto en los logs (user ID, tenant, etc.)
