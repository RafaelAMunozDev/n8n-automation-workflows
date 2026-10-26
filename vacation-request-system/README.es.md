# Sistema de gestión de solicitudes de vacaciones y bajas por enfermedad (n8n)

[English](README.md) · **Español**

## Descripción general

Este proyecto es un flujo de automatización completo hecho con **n8n** que gestiona las solicitudes de ausencia de los empleados, tanto de **días de vacaciones** como de **bajas por enfermedad**.

Simula un sistema real de RR. HH. en el que los empleados envían solicitudes, se aplican validaciones, RR. HH. revisa la solicitud y el sistema procesa automáticamente el resultado.

---

## Funcionalidades

- Envío de solicitudes mediante formulario  
- Normalización y transformación de los datos  
- Validación de las reglas de negocio:
  - Plazo mínimo de antelación  
  - Validación del rango de fechas  
  - Validación de los días disponibles  
- Soporte para dos tipos de solicitud:
  - Vacaciones  
  - Baja por enfermedad  
- Flujo de aprobación de RR. HH. a través de Discord  
- Gestión automática de la decisión:
  - Creación del evento en el calendario  
  - Actualización de la base de datos  
  - Notificaciones por correo electrónico  
- Gestión de errores y respuesta al usuario  

---

## Arquitectura del flujo

El flujo se divide en cuatro fases principales:

### 1. Entrada y preparación de los datos
- Recibe la solicitud desde un formulario  
- Normaliza los campos de entrada  
- Calcula el total de días solicitados  

### 2. Capa de validación
- Valida:
  - Que la solicitud se haga con al menos 7 días de antelación  
  - Que el rango de fechas sea válido  
  - Los días de ausencia disponibles en la base de datos  
- Rechaza las solicitudes no válidas con mensajes de error detallados  

### 3. Proceso de aprobación de RR. HH.
- Envía la solicitud a RR. HH. por Discord  
- Espera la aprobación o el rechazo manual (con el formulario de reanudación de n8n)  
- Incluye la información de contexto (empleado, fechas, tipo)  

### 4. Procesamiento final
- Si se aprueba:
  - Crea el evento en Google Calendar  
  - Actualiza la base de datos (vacation_days o sick_days)  
  - Envía un correo de confirmación  
- Si se rechaza:
  - Envía un correo de rechazo con el motivo  

---

## Estructura de la base de datos

Este flujo usa una tabla SQL para llevar el control de los días de ausencia disponibles:

```sql
CREATE TABLE days_off (
  id SERIAL PRIMARY KEY,
  email TEXT,
  vacation_days SMALLINT,
  sick_days SMALLINT
);
```

### Puntos clave

- Cada empleado se identifica por su correo electrónico  
- El sistema usa el correo indicado en el formulario para:
  - Consultar los días de ausencia disponibles  
  - Actualizar el saldo restante tras la aprobación  
- Se gestionan dos tipos de saldo:
  - `vacation_days`  
  - `sick_days`  

---

## Lógica de negocio

### Cálculo de días

El flujo calcula el número de días solicitados con JavaScript:

- Convierte las fechas para evitar problemas de zona horaria  
- Incluye tanto el día de inicio como el de fin  

```js
Math.ceil((endDate - startDate) / (1000 * 60 * 60 * 24)) + 1
```

### Reglas de validación

- Las solicitudes deben enviarse con al menos 7 días de antelación  
- La fecha de fin debe ser posterior a la de inicio  
- Los días solicitados no pueden superar el saldo disponible  

El sistema elige dinámicamente:
- `vacation_days` o `sick_days` según el tipo de solicitud  

---

## Actualización dinámica de la base de datos

Una única consulta SQL actualiza el campo correcto:

```sql
UPDATE days_off
SET 
  vacation_days = CASE 
    WHEN $3 = 'Vacaciones' THEN vacation_days - $1 
    ELSE vacation_days 
  END,
  sick_days = CASE 
    WHEN $3 = 'Baja por enfermedad' THEN sick_days - $1 
    ELSE sick_days 
  END
WHERE email = $2;
```

---

## Integraciones

Este flujo se integra con:

- **Google Calendar**  
  Crea automáticamente los eventos de las solicitudes aprobadas  

- **Discord**  
  Envía las solicitudes de aprobación a RR. HH.  
  Incluye un enlace para aprobar o rechazar  

- **Correo electrónico**  
  Envía notificaciones de:
  - Errores de validación  
  - Confirmación de la aprobación  
  - Rechazo con el motivo  

- **Base de datos SQL**  
  Guarda y actualiza los saldos de días de ausencia  

---

## Credenciales necesarias

Para ejecutar este flujo tienes que configurar tus propias credenciales:

- API de Google Calendar  
- Base de datos (PostgreSQL / MySQL)  
- Webhook o bot de Discord  
- Servicio de correo electrónico (SMTP / Gmail)  

Por seguridad, las credenciales no se incluyen en el JSON exportado del flujo.

---

## Cómo usarlo

1. Importa el flujo en n8n  
2. Configura todas las credenciales necesarias  
3. Comprueba que la tabla de la base de datos existe  
4. Ejecuta el flujo  
5. Envía una solicitud de ausencia mediante el formulario  

---

## Notas

- El flujo está pensado para simular un sistema real de automatización de RR. HH.  
- Sigue principios de arquitectura limpia:
  - Separación de responsabilidades  
  - Capa de validación clara  
  - Procesamiento estructurado  
- Los nombres internos de los nodos están en español; la documentación está en inglés y en español  

---

## Objetivo

Este proyecto se hizo como práctica con n8n, centrado en resolver un problema de negocio realista mediante la automatización.

---

## Mejoras futuras

- Sistema de roles para varios usuarios (RR. HH. y empleado)  
- Panel web para las solicitudes  
- Integración con el calendario de festivos  
- Integración con Slack  
