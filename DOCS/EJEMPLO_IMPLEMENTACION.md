# Ejemplo Práctico: Implementación Multi-Industria

## 🎯 Caso de Uso: Transformar el Bot Actual

Vamos a mostrar cómo transformar el bot actual de barbería en una plataforma base que pueda servir a múltiples industrias.

---

## 🔧 Refactoring del Código Actual

### **1. Extraer Configuración Hardcodeada**

#### **Antes (Código Actual)**
```typescript
// src/flows/seller.flow.ts
const PROMPT_SELLER = `Eres el asistente virtual en la prestigiosa barbería "Barbería Flow 25", ubicada en Madrid, Plaza de Castilla 4A. Tu principal responsabilidad es responder a las consultas de los clientes y ayudarles a programar sus citas.

FECHA DE HOY: {CURRENT_DAY}

SOBRE "BARBERÍA FLOW 25":
Nos distinguimos por ofrecer cortes de cabello modernos y siempre a la vanguardia. Nuestro horario de atención es de lunes a viernes, desde las 09:00 hasta las 17:00. Para más información, visita nuestro sitio web en "barberflow.co". Aceptamos pagos en efectivo y a través de PayPal. Recuerda que es necesario programar una cita.

PRECIOS DE LOS SERVICIOS:
- Corte de pelo de hombre 10USD
- Corte de pelo + barba 15 USD

HISTORIAL DE CONVERSACIÓN:
--------------
{HISTORIAL_CONVERSACION}
--------------

DIRECTRICES DE INTERACCIÓN:
1. Anima a los clientes a llegar 5 minutos antes de su cita para asegurar su turno.
2. Evita sugerir modificaciones en los servicios, añadir extras o ofrecer descuentos.
3. Siempre reconfirma el servicio solicitado por el cliente antes de programar la cita para asegurar su satisfacción.

Respuesta útil:`;
```

#### **Después (Configuración Dinámica)**
```typescript
// src/config/industries/barbershop.ts
export const barbershopConfig: IndustryConfig = {
  id: 'barbershop',
  name: 'Barbería Flow 25',
  business: {
    name: 'Barbería Flow 25',
    location: 'Madrid, Plaza de Castilla 4A',
    website: 'barberflow.co',
    hours: {
      monday: { start: '09:00', end: '17:00' },
      tuesday: { start: '09:00', end: '17:00' },
      wednesday: { start: '09:00', end: '17:00' },
      thursday: { start: '09:00', end: '17:00' },
      friday: { start: '09:00', end: '17:00' },
      saturday: { start: '09:00', end: '17:00' },
      sunday: { start: 'closed', end: 'closed' }
    },
    services: [
      { name: 'Corte de pelo de hombre', price: 10, currency: 'USD' },
      { name: 'Corte de pelo + barba', price: 15, currency: 'USD' }
    ],
    paymentMethods: ['efectivo', 'paypal']
  },
  prompts: {
    seller: {
      template: `
Eres el asistente virtual en {BUSINESS_NAME}, ubicada en {LOCATION}. Tu principal responsabilidad es responder a las consultas de los clientes y ayudarles a programar sus citas.

FECHA DE HOY: {CURRENT_DAY}

SOBRE "{BUSINESS_NAME}":
{ABOUT_BUSINESS}

HORARIO DE ATENCIÓN:
{HOURS}

SERVICIOS Y PRECIOS:
{SERVICES_LIST}

MÉTODOS DE PAGO:
{PAYMENT_METHODS}

HISTORIAL DE CONVERSACIÓN:
--------------
{HISTORIAL_CONVERSACION}
--------------

DIRECTRICES DE INTERACCIÓN:
{INTERACTION_GUIDELINES}

Respuesta útil:`,
      variables: [
        'BUSINESS_NAME',
        'LOCATION', 
        'ABOUT_BUSINESS',
        'HOURS',
        'SERVICES_LIST',
        'PAYMENT_METHODS',
        'HISTORIAL_CONVERSACION',
        'INTERACTION_GUIDELINES'
      ]
    }
  }
}
```

### **2. Crear Sistema de Templates Dinámicos**

```typescript
// src/services/template.service.ts
export class TemplateService {
  private templates: Map<string, string> = new Map()
  private variables: Map<string, any> = new Map()

  registerTemplate(name: string, template: string) {
    this.templates.set(name, template)
  }

  setVariable(name: string, value: any) {
    this.variables.set(name, value)
  }

  render(templateName: string): string {
    const template = this.templates.get(templateName)
    if (!template) throw new Error(`Template not found: ${templateName}`)

    return template.replace(/\{(\w+)\}/g, (match, variable) => {
      return this.variables.get(variable) || match
    })
  }

  renderWithContext(templateName: string, context: Record<string, any>): string {
    const template = this.templates.get(templateName)
    if (!template) throw new Error(`Template not found: ${templateName}`)

    return template.replace(/\{(\w+)\}/g, (match, variable) => {
      return context[variable] || match
    })
  }
}
```

### **3. Flujo Configurable**

```typescript
// src/flows/configurable.flow.ts
export class ConfigurableFlow {
  constructor(
    private config: FlowConfig,
    private templateService: TemplateService,
    private aiService: AIService
  ) {}

  async execute(ctx: BotContext, methods: BotMethods) {
    const context = await this.buildContext(ctx, methods)
    
    for (const step of this.config.steps) {
      await this.executeStep(step, context, methods)
    }
  }

  private async buildContext(ctx: BotContext, methods: BotMethods) {
    const history = getHistoryParse(methods.state)
    const currentDate = getFullCurrentDate()
    
    return {
      BUSINESS_NAME: this.config.business.name,
      LOCATION: this.config.business.location,
      ABOUT_BUSINESS: this.config.business.about,
      HOURS: this.formatHours(this.config.business.hours),
      SERVICES_LIST: this.formatServices(this.config.business.services),
      PAYMENT_METHODS: this.config.business.paymentMethods.join(', '),
      HISTORIAL_CONVERSACION: history,
      CURRENT_DAY: currentDate,
      INTERACTION_GUIDELINES: this.config.interactionGuidelines
    }
  }

  private async executeStep(step: FlowStep, context: any, methods: BotMethods) {
    switch (step.type) {
      case 'ai_response':
        await this.handleAIResponse(step, context, methods)
        break
      case 'data_collection':
        await this.handleDataCollection(step, context, methods)
        break
      case 'integration_call':
        await this.handleIntegrationCall(step, context, methods)
        break
    }
  }

  private async handleAIResponse(step: AIResponseStep, context: any, methods: BotMethods) {
    const prompt = this.templateService.renderWithContext(step.prompt, context)
    
    const response = await this.aiService.createChat([
      { role: 'system', content: prompt },
      { role: 'user', content: context.userMessage }
    ])

    await handleHistory({ content: response, role: 'assistant' }, methods.state)
    
    const chunks = response.split(/(?<!\d)\.\s+/g)
    for (const chunk of chunks) {
      await methods.flowDynamic([{ 
        body: chunk.trim(), 
        delay: generateTimer(150, 250) 
      }])
    }
  }
}
```

---

## 🏭 Ejemplos de Configuración por Industria

### **1. Bienes Raíces**

```typescript
// src/config/industries/real-estate.ts
export const realEstateConfig: IndustryConfig = {
  id: 'real-estate',
  name: 'Inmobiliaria Premium',
  business: {
    name: 'Inmobiliaria Premium',
    location: 'Madrid, España',
    website: 'inmobiliariapremium.es',
    about: 'Especialistas en propiedades de lujo y exclusivas en Madrid',
    hours: {
      monday: { start: '09:00', end: '18:00' },
      tuesday: { start: '09:00', end: '18:00' },
      wednesday: { start: '09:00', end: '18:00' },
      thursday: { start: '09:00', end: '18:00' },
      friday: { start: '09:00', end: '18:00' },
      saturday: { start: '10:00', end: '14:00' },
      sunday: { start: 'closed', end: 'closed' }
    },
    services: [
      { name: 'Consulta de propiedades', price: 0, currency: 'EUR' },
      { name: 'Visita guiada', price: 0, currency: 'EUR' },
      { name: 'Asesoría inmobiliaria', price: 500, currency: 'EUR' }
    ]
  },
  flows: [
    {
      id: 'property-search',
      name: 'Búsqueda de Propiedades',
      steps: [
        {
          type: 'ai_response',
          prompt: 'property_inquiry',
          variables: ['property_type', 'budget', 'location', 'features']
        },
        {
          type: 'integration_call',
          service: 'property_database',
          action: 'search_properties'
        },
        {
          type: 'ai_response',
          prompt: 'property_results',
          variables: ['properties_list', 'recommendations']
        }
      ]
    },
    {
      id: 'schedule-visit',
      name: 'Programar Visita',
      steps: [
        {
          type: 'data_collection',
          field: 'preferred_date',
          validation: 'date_validation'
        },
        {
          type: 'data_collection',
          field: 'preferred_time',
          validation: 'time_validation'
        },
        {
          type: 'integration_call',
          service: 'agent_scheduler',
          action: 'check_availability'
        },
        {
          type: 'agent_transfer',
          criteria: 'agent_specialization',
          fallback: 'general_agent'
        }
      ]
    }
  ],
  prompts: {
    property_inquiry: {
      template: `
Eres un asistente virtual especializado en bienes raíces para {BUSINESS_NAME}.

INFORMACIÓN DEL NEGOCIO:
- Ubicación: {LOCATION}
- Especialidad: {ABOUT_BUSINESS}
- Horarios: {HOURS}

PROPIEDADES DISPONIBLES:
{PROPERTIES_LIST}

HISTORIAL DE CONVERSACIÓN:
{HISTORIAL_CONVERSACION}

INSTRUCCIONES:
- Ayuda a los clientes a encontrar propiedades que se ajusten a sus necesidades
- Solicita información específica: tipo de propiedad, presupuesto, ubicación preferida
- Proporciona información detallada sobre propiedades disponibles
- Sugiere programar visitas con agentes especializados

Respuesta útil:`,
      variables: [
        'BUSINESS_NAME',
        'LOCATION',
        'ABOUT_BUSINESS', 
        'HOURS',
        'PROPERTIES_LIST',
        'HISTORIAL_CONVERSACION'
      ]
    }
  },
  integrations: [
    {
      name: 'property_database',
      type: 'api',
      endpoint: 'https://api.properties.com/search',
      authentication: 'api_key'
    },
    {
      name: 'agent_scheduler',
      type: 'calendar',
      provider: 'google_calendar',
      scope: 'agent_availability'
    }
  ]
}
```

### **2. Asistencia Técnica**

```typescript
// src/config/industries/tech-support.ts
export const techSupportConfig: IndustryConfig = {
  id: 'tech-support',
  name: 'TechSupport Pro',
  business: {
    name: 'TechSupport Pro',
    location: 'Barcelona, España',
    website: 'techsupportpro.es',
    about: 'Servicios de asistencia técnica especializada para empresas y particulares',
    hours: {
      monday: { start: '08:00', end: '20:00' },
      tuesday: { start: '08:00', end: '20:00' },
      wednesday: { start: '08:00', end: '20:00' },
      thursday: { start: '08:00', end: '20:00' },
      friday: { start: '08:00', end: '20:00' },
      saturday: { start: '09:00', end: '18:00' },
      sunday: { start: '10:00', end: '16:00' }
    },
    services: [
      { name: 'Diagnóstico remoto', price: 25, currency: 'EUR' },
      { name: 'Reparación en sitio', price: 50, currency: 'EUR' },
      { name: 'Mantenimiento preventivo', price: 75, currency: 'EUR' }
    ]
  },
  flows: [
    {
      id: 'issue-diagnosis',
      name: 'Diagnóstico de Problemas',
      steps: [
        {
          type: 'ai_response',
          prompt: 'issue_description',
          variables: ['device_type', 'problem_description', 'error_messages']
        },
        {
          type: 'integration_call',
          service: 'knowledge_base',
          action: 'search_solutions'
        },
        {
          type: 'ai_response',
          prompt: 'diagnosis_result',
          variables: ['possible_solutions', 'escalation_needed']
        }
      ]
    },
    {
      id: 'schedule-technician',
      name: 'Programar Técnico',
      steps: [
        {
          type: 'data_collection',
          field: 'issue_severity',
          validation: 'severity_validation'
        },
        {
          type: 'data_collection',
          field: 'preferred_date',
          validation: 'date_validation'
        },
        {
          type: 'integration_call',
          service: 'technician_scheduler',
          action: 'find_available_technician'
        },
        {
          type: 'agent_transfer',
          criteria: 'technician_specialization',
          fallback: 'general_support'
        }
      ]
    }
  ],
  prompts: {
    issue_description: {
      template: `
Eres un asistente virtual especializado en asistencia técnica para {BUSINESS_NAME}.

INFORMACIÓN DEL NEGOCIO:
- Ubicación: {LOCATION}
- Especialidad: {ABOUT_BUSINESS}
- Horarios: {HOURS}

SERVICIOS DISPONIBLES:
{SERVICES_LIST}

BASE DE CONOCIMIENTOS:
{KNOWLEDGE_BASE}

HISTORIAL DE CONVERSACIÓN:
{HISTORIAL_CONVERSACION}

INSTRUCCIONES:
- Ayuda a diagnosticar problemas técnicos
- Proporciona soluciones básicas cuando sea posible
- Escala a técnicos especializados cuando sea necesario
- Programa visitas técnicas según la urgencia

Respuesta útil:`,
      variables: [
        'BUSINESS_NAME',
        'LOCATION',
        'ABOUT_BUSINESS',
        'HOURS', 
        'SERVICES_LIST',
        'KNOWLEDGE_BASE',
        'HISTORIAL_CONVERSACION'
      ]
    }
  }
}
```

---

## 🔄 Sistema de Plugins

### **1. Plugin de CRM**

```typescript
// src/plugins/crm.plugin.ts
export class CRMPlugin implements Plugin {
  id = 'crm-integration'
  name = 'CRM Integration Plugin'
  version = '1.0.0'
  
  private crmClient: CRMClient

  constructor(config: CRMConfig) {
    this.crmClient = new CRMClient(config)
  }

  async execute(context: PluginContext): Promise<PluginResult> {
    const { customerData, action } = context
    
    switch (action) {
      case 'create_lead':
        return await this.createLead(customerData)
      case 'update_customer':
        return await this.updateCustomer(customerData)
      case 'schedule_followup':
        return await this.scheduleFollowup(customerData)
      default:
        throw new Error(`Unknown action: ${action}`)
    }
  }

  private async createLead(customerData: CustomerData): Promise<PluginResult> {
    const lead = await this.crmClient.createLead({
      name: customerData.name,
      email: customerData.email,
      phone: customerData.phone,
      source: 'whatsapp_bot',
      status: 'new'
    })

    return {
      success: true,
      data: { leadId: lead.id },
      message: 'Lead created successfully'
    }
  }
}
```

### **2. Plugin de Analytics**

```typescript
// src/plugins/analytics.plugin.ts
export class AnalyticsPlugin implements Plugin {
  id = 'analytics'
  name = 'Analytics Plugin'
  version = '1.0.0'

  private analyticsClient: AnalyticsClient

  constructor(config: AnalyticsConfig) {
    this.analyticsClient = new AnalyticsClient(config)
  }

  async execute(context: PluginContext): Promise<PluginResult> {
    const { event, data } = context

    await this.analyticsClient.trackEvent({
      event,
      properties: {
        ...data,
        timestamp: new Date().toISOString(),
        source: 'whatsapp_bot'
      }
    })

    return {
      success: true,
      data: { eventTracked: true },
      message: 'Event tracked successfully'
    }
  }
}
```

---

## 🚀 Implementación Práctica

### **1. Configuración del Bot Principal**

```typescript
// src/app.ts (Refactored)
import 'dotenv/config'
import { createBot, MemoryDB, createProvider } from '@bot-whatsapp/bot'
import { BaileysProvider } from '@bot-whatsapp/provider-baileys'

import { IndustryManager } from './services/industry.manager'
import { TemplateService } from './services/template.service'
import { PluginManager } from './services/plugin.manager'
import AIClass from './services/ai'

// Configuraciones de industrias
import { barbershopConfig } from './config/industries/barbershop'
import { realEstateConfig } from './config/industries/real-estate'
import { techSupportConfig } from './config/industries/tech-support'

const main = async () => {
  // Inicializar servicios
  const templateService = new TemplateService()
  const pluginManager = new PluginManager()
  const industryManager = new IndustryManager(templateService, pluginManager)
  const ai = new AIClass(process.env.OPEN_API_KEY, 'gpt-3.5-turbo-16k')

  // Registrar configuraciones de industrias
  industryManager.registerIndustry(barbershopConfig)
  industryManager.registerIndustry(realEstateConfig)
  industryManager.registerIndustry(techSupportConfig)

  // Registrar plugins
  pluginManager.registerPlugin(new CRMPlugin({
    apiKey: process.env.CRM_API_KEY,
    endpoint: process.env.CRM_ENDPOINT
  }))

  pluginManager.registerPlugin(new AnalyticsPlugin({
    apiKey: process.env.ANALYTICS_API_KEY,
    endpoint: process.env.ANALYTICS_ENDPOINT
  }))

  // Crear bot con configuración dinámica
  const provider = createProvider(BaileysProvider)
  
  await createBot({
    database: new MemoryDB(),
    provider,
    flow: industryManager.getFlows()
  }, { 
    extensions: { 
      ai,
      industryManager,
      templateService,
      pluginManager
    } 
  })
}

main()
```

### **2. Gestor de Industrias**

```typescript
// src/services/industry.manager.ts
export class IndustryManager {
  private industries: Map<string, IndustryConfig> = new Map()
  private flows: Flow[] = []

  constructor(
    private templateService: TemplateService,
    private pluginManager: PluginManager
  ) {}

  registerIndustry(config: IndustryConfig) {
    this.industries.set(config.id, config)
    
    // Registrar templates
    Object.entries(config.prompts).forEach(([name, prompt]) => {
      this.templateService.registerTemplate(name, prompt.template)
    })

    // Crear flujos configurables
    config.flows.forEach(flowConfig => {
      const flow = new ConfigurableFlow(
        flowConfig,
        this.templateService,
        this.pluginManager
      )
      this.flows.push(flow)
    })
  }

  getFlows(): Flow[] {
    return this.flows
  }

  getIndustryConfig(industryId: string): IndustryConfig | undefined {
    return this.industries.get(industryId)
  }
}
```

---

## 📊 Dashboard de Configuración

### **1. Interfaz de Usuario**

```typescript
// src/dashboard/components/IndustryConfigurator.tsx
interface IndustryConfiguratorProps {
  industryId: string
  onConfigChange: (config: IndustryConfig) => void
}

export const IndustryConfigurator: React.FC<IndustryConfiguratorProps> = ({
  industryId,
  onConfigChange
}) => {
  const [config, setConfig] = useState<IndustryConfig>()

  const updateBusinessInfo = (businessInfo: BusinessInfo) => {
    const updatedConfig = { ...config, business: businessInfo }
    setConfig(updatedConfig)
    onConfigChange(updatedConfig)
  }

  const updatePrompts = (prompts: PromptConfig[]) => {
    const updatedConfig = { ...config, prompts }
    setConfig(updatedConfig)
    onConfigChange(updatedConfig)
  }

  return (
    <div className="industry-configurator">
      <BusinessInfoEditor 
        businessInfo={config?.business}
        onUpdate={updateBusinessInfo}
      />
      <PromptEditor 
        prompts={config?.prompts}
        onUpdate={updatePrompts}
      />
      <FlowEditor 
        flows={config?.flows}
        onUpdate={(flows) => {
          const updatedConfig = { ...config, flows }
          setConfig(updatedConfig)
          onConfigChange(updatedConfig)
        }}
      />
    </div>
  )
}
```

---

## 🎯 Beneficios de esta Arquitectura

### **1. Reutilización de Código**
- Core framework compartido
- Plugins reutilizables
- Templates configurables

### **2. Time-to-Market Rápido**
- Configuraciones predefinidas
- Setup en minutos, no semanas
- Personalización sin código

### **3. Escalabilidad**
- Multi-tenant architecture
- Plugins modulares
- Integraciones extensibles

### **4. Mantenibilidad**
- Configuración centralizada
- Testing automatizado
- Deployment simplificado

---

*Ejemplo de implementación generado el: $(date)*
*Versión: 1.0.0*

