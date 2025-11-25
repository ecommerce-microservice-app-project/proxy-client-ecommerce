# Proxy Client (Frontend Application).

##  Descripción

Proxy Client es una **aplicación web frontend** que actúa como cliente para consumir los microservicios backend. Está construida con **Spring Boot**, **Thymeleaf** (para renderizado de páginas HTML) y **OpenFeign** (para comunicación REST con los microservicios).

##  Propósito

- **Interfaz Web**: Proporciona una interfaz de usuario web (páginas HTML)
- **Orquestación**: Combina llamadas a múltiples microservicios para operaciones complejas
- **Autenticación**: Maneja autenticación JWT y sesiones de usuario
- **Capa de Presentación**: Renderiza vistas usando Thymeleaf templates

##  Arquitectura

```
┌─────────────┐
│   Usuario   │
│  (Browser)  │
└──────┬──────┘
       │ HTTP Request
       │
       ▼
┌─────────────┐      ┌─────────────┐      ┌──────────────┐
│ Proxy       │─────>│   Eureka    │─────>│ Product Svc  │
│ Client      │      │ (discovers) │      │ Order Svc    │
│ /app/**     │      └─────────────┘      │ User Svc     │
└─────────────┘                            └──────────────┘
       │
       │ también puede llamar directamente
       │
       └──────────────────────────────────────┘
```

##  Configuración

### Context Path
- **Context Path**: `/app`
- **Puerto**: `8900` (desarrollo local)
- **URL Local**: `http://localhost:8900/app`
- **URL Kubernetes**: `http://proxy-client.ecommerce-dev.svc.cluster.local:8900/app`
- **URL a través de API Gateway**: `http://20.15.17.8:8080/app/**`

### Application Name
- **Nombre**: `PROXY-CLIENT`

## 🔌 Endpoints Disponibles

Todos los endpoints están bajo el context path `/app`:

```
/app/** - Todas las rutas del frontend
```

### Servicios Cliente (OpenFeign)

El Proxy Client usa OpenFeign para comunicarse con los microservicios:

- **ProductClientService**: Comunicación con Product Service
- **OrderClientService**: Comunicación con Order Service
- **CartClientService**: Comunicación con Cart Service
- **UserClientService**: Comunicación con User Service
- **PaymentClientService**: Comunicación con Payment Service

##  Integración con Microservicios

El Proxy Client usa Eureka para descubrir servicios y luego hace llamadas directas usando OpenFeign:

```java
@FeignClient(name = "PRODUCT-SERVICE")
public interface ProductClientService {
    @GetMapping("/product-service/api/products")
    ResponseEntity<DtoCollectionResponse<ProductDto>> findAll();
}
```

Eureka resuelve `PRODUCT-SERVICE` a la URL real del servicio.

##  Tecnologías Utilizadas

- **Spring Boot**: Framework de la aplicación
- **Thymeleaf**: Motor de plantillas para renderizar HTML
- **Spring Security**: Autenticación y autorización
- **OpenFeign**: Cliente REST declarativo
- **JWT**: Tokens para autenticación
- **Spring Cloud Eureka Client**: Descubrimiento de servicios

##  Despliegue

### Desarrollo Local

```bash
./mvnw spring-boot:run
```

Servicio disponible en: `http://localhost:8900/app`

### Docker

```bash
docker build -t proxy-client:0.1.0 .
docker run -p 8900:8900 proxy-client:0.1.0
```

### Kubernetes

El servicio se despliega automáticamente mediante el pipeline CI/CD en el namespace `ecommerce-dev`.

**Tipo de Servicio**: `ClusterIP` (interno, accesible a través de API Gateway)

## Notas Importantes

### Diferencia con API Gateway

- **API Gateway**: Enruta peticiones REST directamente a microservicios
- **Proxy Client**: Aplicación web completa con UI, que internamente llama a microservicios

### Orquestación de Servicios

El Proxy Client puede hacer **orquestación**, es decir, combinar múltiples llamadas:

```
Usuario → /app/checkout
  ↓
Proxy Client:
  1. Llama a Order Service (crear orden)
  2. Llama a Payment Service (procesar pago)
  3. Llama a Shipping Service (crear envío)
  4. Retorna resultado combinado al usuario
```

### Estrategia de Despliegue

- **Namespace**: Siempre `ecommerce-dev` (mismo para dev/stage/prod)
- **Tags de Imagen**:
  - `dev-latest` (branches dev/develop)
  - `stage-latest` (branch stage)
  - `prod-0.1.0` (branches main/master)
- **Tipo de Servicio**: ClusterIP (accesible vía API Gateway en `/app/**`)
- **Replicas**: 1

### Orden de Arranque

El Proxy Client debe iniciar después de:
1. Service Discovery
2. Cloud Config (opcional)
3. Microservicios de negocio (Product, Order, User)
4. API Gateway

**Razón**: Necesita que todos los servicios estén disponibles para poder hacer llamadas.

### Acceso al Frontend

A través del API Gateway:
```
http://20.15.17.8:8080/app/...
```

Directo (desde dentro del cluster):
```
http://proxy-client.ecommerce-dev.svc.cluster.local:8900/app/...
```

##  Testing

Este servicio tiene un intento de pruebas unitarias pero puede ejecutarse sin ellas (configurado con `|| true` en el pipeline).

Las pruebas E2E del frontend se pueden hacer manualmente accediendo a las páginas web renderizadas.
