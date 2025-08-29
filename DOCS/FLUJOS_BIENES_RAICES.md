# Flujos Específicos - Bienes Raíces

## 🏠 Flujos de Conversación Detallados

### **Flujo 1: Agendar Cita de Visita**

```
Usuario: "Quiero agendar una cita"

Bot: "¡Perfecto! Te ayudo a agendar tu visita. Para encontrar el mejor horario para ti, necesito algunos datos:"

Bot: "¿Cuál es tu nombre completo?"
Usuario: "Juan Pérez"

Bot: "¿Cuál es tu número de teléfono?"
Usuario: "5512345678"

Bot: "¿Cuál es tu correo electrónico?"
Usuario: "juan@email.com"

Bot: "¿Qué tipo de propiedad te interesa?"
Opciones: [Departamento 1BR, Departamento 2BR, Departamento 3BR, Penthouse, Aún no sé]
Usuario: "Departamento 2BR"

Bot: "¿Qué fecha te funciona mejor para la visita?"
[Selector de fecha]
Usuario: "15 de enero"

Bot: "¿Qué horario prefieres?"
Opciones: [9:00 AM, 11:00 AM, 2:00 PM, 4:00 PM, 6:00 PM]
Usuario: "2:00 PM"

Bot: "Perfecto, Juan. Tu cita está programada para el 15 de enero a las 2:00 PM. Te enviaré una confirmación por WhatsApp y email. ¿Hay algo más que te gustaría saber sobre el proyecto?"
```

### **Flujo 2: Recorrido Virtual**

```
Usuario: "Quiero ver el recorrido virtual"

Bot: "¡Excelente elección! Tenemos varios recorridos virtuales para que conozcas nuestro proyecto desde la comodidad de tu casa:"

Bot: "¿Qué tipo de recorrido te gustaría ver?"
Opciones: [
  "🏠 Tour Completo del Proyecto (360°)",
  "🏢 Tour de Amenidades", 
  "📱 Tour de Departamentos Modelo",
  "🚁 Vista Aérea (Drone)",
  "📹 Video Promocional"
]
Usuario: "Tour Completo del Proyecto"

Bot: "Aquí tienes el enlace para el Tour Completo del Proyecto:

🔗 https://tour.residencialpalmas.com/completo
⏱️ Duración: 8 minutos

¿Te gustaría que te envíe más información sobre alguna área específica?"
```

### **Flujo 3: Solicitar PDF**

```
Usuario: "Quiero información en PDF"

Bot: "Tenemos varios materiales informativos que pueden ayudarte a conocer mejor nuestro proyecto:"

Bot: "¿Qué información te interesa más?"
Opciones: [
  "📋 Brochure General del Proyecto",
  "💰 Información de Precios y Financiamiento", 
  "🏗️ Planos y Distribuciones",
  "📍 Ubicación y Entorno",
  "🏊‍♂️ Amenidades y Servicios",
  "📞 Información de Contacto"
]
Usuario: "Brochure General"

Bot: "Para enviarte el Brochure General del Proyecto, necesito tu información de contacto:"

Bot: "¿Cuál es tu correo electrónico?"
Usuario: "juan@email.com"

Bot: "¡Perfecto! Te he enviado el Brochure General del Proyecto a juan@email.com. También te lo comparto por WhatsApp.

¿Te gustaría agendar una visita para conocer el proyecto en persona?"
```

### **Flujo 4: Unidades Disponibles**

```
Usuario: "¿Qué unidades tienen disponibles?"

Bot: "Te ayudo a encontrar las unidades disponibles. Actualmente tenemos 45 unidades en venta."

Bot: "¿Qué tipo de propiedad te interesa?"
Opciones: [
  "🏠 Departamentos 1BR",
  "🏢 Departamentos 2BR",
  "🏘️ Departamentos 3BR", 
  "🏰 Penthouse",
  "Ver todas las opciones"
]
Usuario: "Departamentos 2BR"

Bot: "¿En qué rango de precio estás interesado?"
Opciones: [
  "Hasta $2,000,000 MXN",
  "$2,000,000 - $3,000,000 MXN",
  "$3,000,000 - $4,000,000 MXN",
  "$4,000,000 - $5,000,000 MXN",
  "Más de $5,000,000 MXN",
  "Ver todos los precios"
]
Usuario: "$2,000,000 - $3,000,000 MXN"

Bot: "Aquí tienes las unidades disponibles que coinciden con tus criterios:

🏢 Departamento 2BR - Torre A, Piso 8
• 110 m², 2 recámaras, 2 baños
• Precio: $2,450,000 MXN
• Vista: Ciudad

🏢 Departamento 2BR - Torre B, Piso 12  
• 125 m², 2 recámaras, 2 baños
• Precio: $2,850,000 MXN
• Vista: Jardines

¿Te interesa alguna unidad en particular?"
```

### **Flujo 5: Amenidades**

```
Usuario: "¿Qué amenidades tienen?"

Bot: "¡Nuestro proyecto cuenta con amenidades de lujo! ¿Qué te interesa conocer?"

Bot: "Selecciona una categoría:"
Opciones: [
  "🏊‍♂️ Recreación y Deporte",
  "🏋️‍♂️ Gimnasio y Wellness", 
  "🍽️ Restaurantes y Cafés",
  "🚗 Estacionamiento y Seguridad",
  "👨‍👩‍👧‍👦 Áreas Comunes",
  "🌿 Jardines y Exteriores"
]
Usuario: "Recreación y Deporte"

Bot: "En la categoría Recreación y Deporte tenemos:

🏊‍♂️ Alberca infinita con vista panorámica
🎾 Cancha de tenis profesional  
🏃‍♂️ Pista de jogging
🏋️‍♂️ Gimnasio equipado 24/7
🎯 Salón de eventos y fiestas

¿Te gustaría ver fotos de alguna amenidad específica?"
```

### **Flujo 6: Preguntas Generales**

```
Usuario: "¿Cuándo entregan las unidades?"

Bot: "🏗️ **Fechas de Entrega**

Nuestro proyecto tiene entregas escalonadas:

📅 **Torre A**: Diciembre 2024
📅 **Torre B**: Marzo 2025  
📅 **Torre C**: Junio 2025

El proceso de entrega incluye:
• Inspección final de calidad
• Entrega de llaves
• Documentación notarial
• Servicios básicos funcionando

¿Te gustaría saber más sobre el proceso de entrega o agendar una visita?"
```

---

## 🔄 Integraciones Make.com Específicas

### **Escenario 1: Crear Cita en Google Calendar**

```typescript
const calendarScenario = {
  trigger: "appointment_confirmed",
  actions: [
    {
      type: "google_calendar",
      action: "create_event",
      data: {
        title: "Visita Residencial Las Palmas - Juan Pérez",
        description: `
Cliente: Juan Pérez
Teléfono: 5512345678
Email: juan@email.com
Interés: Departamento 2BR
Precio: $2,000,000 - $3,000,000 MXN

Proyecto: Residencial Las Palmas
Dirección: Av. Principal 123, CDMX
Asesor: María González
        `,
        start: "2024-01-15T14:00:00",
        end: "2024-01-15T15:00:00",
        attendees: ["maria@residencialpalmas.com", "juan@email.com"]
      }
    }
  ]
};
```

### **Escenario 2: Crear Lead en CRM**

```typescript
const crmScenario = {
  trigger: "new_lead_created",
  actions: [
    {
      type: "crm",
      action: "create_lead",
      data: {
        name: "Juan Pérez",
        email: "juan@email.com",
        phone: "5512345678",
        source: "whatsapp_chatbot",
        project: "Residencial Las Palmas",
        interest: "Departamento 2BR",
        price_range: "$2,000,000 - $3,000,000 MXN",
        status: "new"
      }
    }
  ]
};
```

### **Escenario 3: Enviar PDF por Email**

```typescript
const pdfEmailScenario = {
  trigger: "pdf_requested",
  actions: [
    {
      type: "email",
      action: "send_pdf",
      data: {
        to: "juan@email.com",
        subject: "Brochure Residencial Las Palmas",
        template: "pdf_delivery",
        attachments: ["brochure_general.pdf"],
        variables: {
          customer_name: "Juan Pérez",
          project_name: "Residencial Las Palmas",
          pdf_type: "Brochure General"
        }
      }
    }
  ]
};
```

---

## 📊 Métricas de Conversión

### **Embudo de Conversión Típico**

```
Conversaciones Iniciadas: 1,000
├── Leads Calificados: 450 (45%)
├── Citas Programadas: 230 (23%)
├── Citas Atendidas: 180 (18%)
└── Ventas Cerradas: 45 (4.5%)
```

### **KPIs por Flujo**

- **Agendar Cita**: 23% conversión
- **Recorrido Virtual**: 15% solicita cita después
- **PDF Informativo**: 18% solicita cita después
- **Unidades Disponibles**: 25% solicita cita después
- **Amenidades**: 12% solicita cita después

---

*Flujos Bienes Raíces generado el: $(date)*
*Versión: 1.0.0*

