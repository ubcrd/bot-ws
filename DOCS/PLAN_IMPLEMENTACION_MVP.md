# Plan de Implementación - MVP Bienes Raíces

## 🎯 **Objetivo del MVP**

Crear un chatbot especializado para bienes raíces con las 6 funcionalidades críticas:
1. Agendar Cita de Visita
2. Ver Recorrido Virtual
3. Solicitar PDF Informativo
4. Información de Unidades Disponibles
5. Información de Amenidades
6. Preguntas Generales del Proyecto

---

## 📋 **Fase 1: Preparación y Configuración (Semana 1)**

### **1.1 Configuración del Entorno de Desarrollo**

#### **Requisitos Previos:**
- [ ] Node.js 18+ instalado
- [ ] pnpm instalado
- [ ] Git configurado
- [ ] Editor de código (VS Code recomendado)

#### **Configuración del Proyecto:**
```bash
# Clonar o inicializar el proyecto
git clone <repo-url> bot-ws-real-estate
cd bot-ws-real-estate

# Instalar dependencias
pnpm install

# Configurar variables de entorno
cp .env.example .env
```

#### **Variables de Entorno Necesarias:**
```env
# Bot Framework
BOT_TOKEN=your_bot_token
PROVIDER_TOKEN=your_provider_token

# OpenAI
OPENAI_API_KEY=your_openai_key

# Base de Datos
DATABASE_URL=your_database_url

# Servicios Externos
GOOGLE_CALENDAR_ID=your_calendar_id
GOOGLE_SERVICE_ACCOUNT=path_to_service_account.json

# Make.com
MAKE_WEBHOOK_URL=your_make_webhook_url

# Email
SMTP_HOST=your_smtp_host
SMTP_PORT=587
SMTP_USER=your_email
SMTP_PASS=your_password
```

### **1.2 Estructura del Proyecto**

```
src/
├── app.ts                          # Punto de entrada principal
├── config/
│   ├── database.ts                 # Configuración de base de datos
│   ├── openai.ts                   # Configuración de OpenAI
│   └── real-estate.ts              # Configuración específica del sector
├── flows/
│   ├── index.ts                    # Exportación de todos los flujos
│   ├── welcome.flow.ts             # Flujo de bienvenida
│   ├── appointment.flow.ts         # Flujo de agendar cita
│   ├── virtual-tour.flow.ts        # Flujo de recorrido virtual
│   ├── pdf-request.flow.ts         # Flujo de solicitar PDF
│   ├── units.flow.ts               # Flujo de unidades disponibles
│   ├── amenities.flow.ts           # Flujo de amenidades
│   └── general-questions.flow.ts   # Flujo de preguntas generales
├── services/
│   ├── ai/
│   │   └── index.ts                # Servicio de IA
│   ├── calendar/
│   │   └── index.ts                # Servicio de calendario
│   ├── crm/
│   │   └── index.ts                # Servicio de CRM
│   ├── email/
│   │   └── index.ts                # Servicio de email
│   └── real-estate/
│       ├── project.ts              # Gestión de proyectos
│       ├── units.ts                # Gestión de unidades
│       └── amenities.ts            # Gestión de amenidades
├── types/
│   └── real-estate.ts              # Tipos específicos del sector
└── utils/
    ├── prompts.ts                  # Prompts especializados
    └── validators.ts               # Validadores de datos
```

---

## 🚀 **Fase 2: Implementación Core (Semana 2-3)**

### **2.1 Configuración Base del Bot**

#### **Paso 1: Configurar el Bot Principal**
```typescript
// src/app.ts
import 'dotenv/config'
import { createBot, MemoryDB, createProvider } from '@bot-whatsapp/bot'
import { BaileysProvider } from '@bot-whatsapp/provider-baileys'
import AIClass from './services/ai'
import flows from './flows'
import { realEstateConfig } from './config/real-estate'

const ai = new AIClass(process.env.OPENAI_API_KEY, 'gpt-3.5-turbo-16k')

const main = async () => {
    const provider = createProvider(BaileysProvider)
    
    await createBot({
        database: new MemoryDB(),
        provider,
        flow: flows
    }, { 
        extensions: { 
            ai,
            realEstate: realEstateConfig
        } 
    })
}

main()
```

#### **Paso 2: Configuración Específica del Sector**
```typescript
// src/config/real-estate.ts
export const realEstateConfig = {
    project: {
        name: "Residencial Las Palmas",
        description: "Proyecto residencial de lujo en el corazón de la ciudad",
        developer: "Constructora ABC",
        location: {
            address: "Av. Principal 123",
            city: "Ciudad de México",
            state: "CDMX"
        },
        contact: {
            phone: "+52 55 1234 5678",
            email: "info@residencialpalmas.com",
            website: "https://residencialpalmas.com"
        }
    },
    propertyTypes: [
        {
            id: "1br",
            name: "Departamento 1BR",
            size: { min: 85, max: 95 },
            priceRange: { min: 1500000, max: 2000000 }
        },
        {
            id: "2br", 
            name: "Departamento 2BR",
            size: { min: 110, max: 130 },
            priceRange: { min: 2000000, max: 3000000 }
        }
        // ... más tipos
    ],
    amenities: [
        {
            id: "pool",
            name: "Alberca",
            category: "Recreación",
            icon: "🏊‍♂️"
        }
        // ... más amenidades
    ]
}
```

### **2.2 Implementación de Flujos**

#### **Paso 3: Flujo de Bienvenida**
```typescript
// src/flows/welcome.flow.ts
import { createFlow, addKeyword, EVENTS } from '@bot-whatsapp/bot'
import { realEstatePrompts } from '../utils/prompts'

export const welcomeFlow = createFlow([
    addKeyword(EVENTS.WELCOME)
        .addAnswer(realEstatePrompts.welcome, {
            buttons: [
                { body: '📅 Agendar Cita' },
                { body: '🏠 Recorrido Virtual' },
                { body: '📋 Solicitar PDF' },
                { body: '🏢 Unidades Disponibles' },
                { body: '🏊‍♂️ Amenidades' },
                { body: '❓ Preguntas Generales' }
            ]
        })
])
```

#### **Paso 4: Flujo de Agendar Cita**
```typescript
// src/flows/appointment.flow.ts
import { createFlow, addKeyword } from '@bot-whatsapp/bot'
import { appointmentService } from '../services/real-estate/appointment'

export const appointmentFlow = createFlow([
    addKeyword(['agendar', 'cita', 'visita'])
        .addAnswer('¡Perfecto! Te ayudo a agendar tu visita. ¿Cuál es tu nombre completo?')
        .addAnswer(
            { capture: true },
            async (ctx, { flowDynamic, state }) => {
                await state.update({ customerName: ctx.body })
                await flowDynamic('¿Cuál es tu número de teléfono?')
            }
        )
        .addAnswer(
            { capture: true },
            async (ctx, { flowDynamic, state }) => {
                await state.update({ customerPhone: ctx.body })
                await flowDynamic('¿Cuál es tu correo electrónico?')
            }
        )
        // ... continuar con más pasos
])
```

### **2.3 Servicios Especializados**

#### **Paso 5: Servicio de Proyectos**
```typescript
// src/services/real-estate/project.ts
export class ProjectService {
    async getProjectInfo() {
        return {
            name: "Residencial Las Palmas",
            description: "Proyecto residencial de lujo...",
            // ... más información
        }
    }

    async getAvailableUnits(filters: any) {
        // Lógica para obtener unidades disponibles
        return []
    }

    async getAmenities(category?: string) {
        // Lógica para obtener amenidades
        return []
    }
}
```

---

## 🔧 **Fase 3: Integraciones (Semana 4)**

### **3.1 Integración con Make.com**

#### **Paso 6: Configurar Webhooks**
```typescript
// src/services/integrations/make.ts
export class MakeIntegration {
    private webhookUrl: string

    constructor() {
        this.webhookUrl = process.env.MAKE_WEBHOOK_URL
    }

    async createAppointment(data: any) {
        const response = await fetch(this.webhookUrl, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                action: 'create_appointment',
                data: data
            })
        })
        return response.json()
    }

    async sendPDF(data: any) {
        // Lógica para enviar PDF
    }

    async createLead(data: any) {
        // Lógica para crear lead
    }
}
```

### **3.2 Integración con Google Calendar**

#### **Paso 7: Configurar Google Calendar**
```typescript
// src/services/calendar/index.ts
import { google } from 'googleapis'

export class CalendarService {
    private calendar

    constructor() {
        const auth = new google.auth.GoogleAuth({
            keyFile: process.env.GOOGLE_SERVICE_ACCOUNT,
            scopes: ['https://www.googleapis.com/auth/calendar']
        })
        
        this.calendar = google.calendar({ version: 'v3', auth })
    }

    async createAppointment(appointmentData: any) {
        const event = {
            summary: `Visita ${appointmentData.project} - ${appointmentData.customerName}`,
            description: appointmentData.description,
            start: {
                dateTime: appointmentData.startTime,
                timeZone: 'America/Mexico_City'
            },
            end: {
                dateTime: appointmentData.endTime,
                timeZone: 'America/Mexico_City'
            },
            attendees: appointmentData.attendees
        }

        return await this.calendar.events.insert({
            calendarId: process.env.GOOGLE_CALENDAR_ID,
            resource: event
        })
    }
}
```

---

## 🧪 **Fase 4: Testing y Validación (Semana 5)**

### **4.1 Testing de Flujos**

#### **Paso 8: Crear Tests Unitarios**
```typescript
// tests/flows/appointment.test.ts
import { appointmentFlow } from '../../src/flows/appointment.flow'

describe('Appointment Flow', () => {
    test('should collect customer information', async () => {
        // Test del flujo de agendar cita
    })

    test('should validate phone number', async () => {
        // Test de validación
    })

    test('should create calendar event', async () => {
        // Test de integración
    })
})
```

### **4.2 Testing de Integraciones**

#### **Paso 9: Testing de Make.com**
```typescript
// tests/integrations/make.test.ts
import { MakeIntegration } from '../../src/services/integrations/make'

describe('Make.com Integration', () => {
    test('should create appointment', async () => {
        const make = new MakeIntegration()
        const result = await make.createAppointment({
            customerName: 'Juan Pérez',
            customerPhone: '5512345678',
            customerEmail: 'juan@email.com',
            appointmentDate: '2024-01-15',
            appointmentTime: '14:00'
        })
        
        expect(result.success).toBe(true)
    })
})
```

---

## 🚀 **Fase 5: Despliegue (Semana 6)**

### **5.1 Configuración de Producción**

#### **Paso 10: Configurar Servidor**
```bash
# Configurar variables de producción
export NODE_ENV=production
export DATABASE_URL=production_database_url
export OPENAI_API_KEY=production_openai_key

# Instalar dependencias de producción
pnpm install --production

# Construir el proyecto
pnpm build
```

### **5.2 Despliegue en Railway/Render**

#### **Paso 11: Configurar CI/CD**
```yaml
# .github/workflows/deploy.yml
name: Deploy to Railway

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v2
        with:
          node-version: '18'
      - run: npm install -g pnpm
      - run: pnpm install
      - run: pnpm build
      - run: pnpm test
      - uses: railway/deploy@v1
        with:
          railway_token: ${{ secrets.RAILWAY_TOKEN }}
```

---

## 📊 **Fase 6: Monitoreo y Optimización (Semana 7+)**

### **6.1 Métricas y Analytics**

#### **Paso 12: Implementar Tracking**
```typescript
// src/services/analytics/index.ts
export class AnalyticsService {
    async trackConversation(data: any) {
        // Tracking de conversaciones
    }

    async trackConversion(data: any) {
        // Tracking de conversiones
    }

    async generateReport() {
        // Generar reportes
    }
}
```

---

## 🎯 **Checklist de Implementación**

### **Semana 1 - Preparación:**
- [ ] Configurar entorno de desarrollo
- [ ] Instalar dependencias
- [ ] Configurar variables de entorno
- [ ] Crear estructura de carpetas
- [ ] Configurar Git y repositorio

### **Semana 2-3 - Core:**
- [ ] Implementar flujo de bienvenida
- [ ] Implementar flujo de agendar cita
- [ ] Implementar flujo de recorrido virtual
- [ ] Implementar flujo de solicitar PDF
- [ ] Implementar flujo de unidades disponibles
- [ ] Implementar flujo de amenidades
- [ ] Implementar flujo de preguntas generales

### **Semana 4 - Integraciones:**
- [ ] Configurar Make.com webhooks
- [ ] Integrar Google Calendar
- [ ] Configurar servicio de email
- [ ] Implementar CRM básico
- [ ] Configurar base de datos

### **Semana 5 - Testing:**
- [ ] Crear tests unitarios
- [ ] Crear tests de integración
- [ ] Testing de flujos completos
- [ ] Validación de prompts
- [ ] Testing de performance

### **Semana 6 - Despliegue:**
- [ ] Configurar servidor de producción
- [ ] Configurar CI/CD
- [ ] Desplegar en Railway/Render
- [ ] Configurar monitoreo
- [ ] Testing en producción

### **Semana 7+ - Optimización:**
- [ ] Implementar analytics
- [ ] Optimizar prompts
- [ ] Mejorar flujos basado en feedback
- [ ] Implementar métricas
- [ ] Documentación final

---

## 🚨 **Riesgos y Mitigaciones**

### **Riesgos Técnicos:**
- **Riesgo**: Problemas con la API de OpenAI
  - **Mitigación**: Implementar fallbacks y rate limiting

- **Riesgo**: Problemas con Make.com
  - **Mitigación**: Implementar retry logic y webhooks alternativos

- **Riesgo**: Problemas de performance
  - **Mitigación**: Implementar caching y optimización de prompts

### **Riesgos de Negocio:**
- **Riesgo**: Baja tasa de conversión
  - **Mitigación**: A/B testing de prompts y flujos

- **Riesgo**: Problemas de escalabilidad
  - **Mitigación**: Arquitectura modular y microservicios

---

## 📈 **Métricas de Éxito**

### **Métricas Técnicas:**
- Tiempo de respuesta < 2 segundos
- Uptime > 99.9%
- Tasa de error < 1%

### **Métricas de Negocio:**
- Tasa de conversión > 20%
- Leads calificados > 50%
- Satisfacción del cliente > 4.5/5

---

*Plan de Implementación generado el: $(date)*
*Versión: 1.0.0*
