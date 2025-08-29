# Arquitectura Técnica - Bot WhatsApp

## 🏗️ Diagrama de Arquitectura

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   WhatsApp      │    │   Telegram      │    │   Otros         │
│   (Baileys)     │    │   (Provider)    │    │   Providers     │
└─────────┬───────┘    └─────────┬───────┘    └─────────┬───────┘
          │                      │                      │
          └──────────────────────┼──────────────────────┘
                                 │
                    ┌─────────────▼─────────────┐
                    │    @bot-whatsapp/bot      │
                    │    (Framework Core)       │
                    └─────────────┬─────────────┘
                                 │
                    ┌─────────────▼─────────────┐
                    │      Flow Router          │
                    │   (Main Layer - AI)       │
                    └─────────────┬─────────────┘
                                 │
          ┌──────────────────────┼──────────────────────┐
          │                      │                      │
┌─────────▼─────────┐  ┌─────────▼─────────┐  ┌─────────▼─────────┐
│   Welcome Flow    │  │   Seller Flow     │  │  Schedule Flow    │
│   (Entry Point)   │  │   (Business Info) │  │   (Booking)       │
└─────────┬─────────┘  └─────────┬─────────┘  └─────────┬─────────┘
          │                      │                      │
          └──────────────────────┼──────────────────────┘
                                 │
                    ┌─────────────▼─────────────┐
                    │    Confirm Flow           │
                    │   (Data Collection)       │
                    └─────────────┬─────────────┘
                                 │
                    ┌─────────────▼─────────────┐
                    │    State Management       │
                    │   (MemoryDB)              │
                    └─────────────┬─────────────┘
                                 │
          ┌──────────────────────┼──────────────────────┐
          │                      │                      │
┌─────────▼─────────┐  ┌─────────▼─────────┐  ┌─────────▼─────────┐
│   OpenAI Service  │  │  Calendar Service │  │  History Utils    │
│   (GPT-3.5/4)     │  │   (Make.com)      │  │   (State Mgmt)    │
└───────────────────┘  └───────────────────┘  └───────────────────┘
```

## 🔧 Componentes del Sistema

### 1. **Core Framework (@bot-whatsapp/bot)**

**Responsabilidades:**
- Gestión de conexiones con proveedores de mensajería
- Routing de mensajes a flujos apropiados
- Gestión de estado de conversaciones
- Manejo de eventos y acciones

**Configuración:**
```typescript
await createBot({
    database: new MemoryDB(),        // Almacenamiento en memoria
    provider,                        // Proveedor de mensajería
    flow: flows                      // Flujos de conversación
}, { extensions: { ai } })           // Extensiones (IA)
```

### 2. **Providers de Mensajería**

#### **BaileysProvider (WhatsApp)**
- **Tecnología**: Baileys (librería no oficial de WhatsApp)
- **Ventajas**: Gratuito, sin límites de API
- **Desventajas**: Requiere QR code, puede romperse con updates
- **Uso**: `createProvider(BaileysProvider)`

#### **TelegramProvider**
- **Tecnología**: Bot API oficial de Telegram
- **Ventajas**: Estable, documentación oficial
- **Desventajas**: Requiere token, límites de API
- **Uso**: `createProvider(TelegramProvider, { token })`

### 3. **Sistema de Flujos**

#### **Arquitectura de Flujos**
```typescript
// Estructura de un flujo
const flow = addKeyword(EVENTS.WELCOME)
    .addAction(async (ctx, { state, flowDynamic }) => {
        // Lógica del flujo
    })
    .addAction({ capture: true }, async (ctx, { state }) => {
        // Captura de datos del usuario
    })
```

#### **Tipos de Eventos**
- `EVENTS.WELCOME`: Mensaje inicial
- `EVENTS.ACTION`: Acción específica
- `EVENTS.MEDIA`: Archivos multimedia
- `EVENTS.LOCATION`: Ubicación

### 4. **Capas de Procesamiento**

#### **Conversational Layer**
```typescript
// Almacena historial de conversación
export default async ({ body }: BotContext, { state }: BotMethods) => {
    await handleHistory({ content: body, role: 'user' }, state)
}
```

**Funcionalidades:**
- Captura de mensajes del usuario
- Almacenamiento en estado
- Mantenimiento de contexto

#### **Main Layer (Router Inteligente)**
```typescript
// Determina flujo basado en IA
const prompt = `Analiza el contexto y determina la acción:
1. AGENDAR: Programar cita
2. HABLAR: Información del negocio
3. CONFIRMAR: Confirmar datos`
```

**Funcionalidades:**
- Análisis de intención con IA
- Routing dinámico a flujos
- Contexto conversacional

### 5. **Servicios Externos**

#### **AI Service (OpenAI)**
```typescript
class AIClass {
    private openai: OpenAI;
    private model: string;

    createChat = async (messages: ChatCompletionMessageParam[]) => {
        const completion = await this.openai.chat.completions.create({
            model: this.model,
            messages,
            temperature: 0,
            max_tokens: 326
        });
        return completion.choices[0].message.content;
    };
}
```

**Configuración:**
- **Modelo por defecto**: `gpt-3.5-turbo-16k`
- **Modelo para decisiones críticas**: `gpt-4`
- **Parámetros**: temperature=0, max_tokens=326

#### **Calendar Service (Make.com)**
```typescript
// Consulta disponibilidad
const getCurrentCalendar = async (): Promise<string> => {
    const dataCalendarApi = await fetch('https://hook.eu2.make.com/yvwkwwxs82vw3o23j7ndtv3luhtvucus')
    const json: any[] = await dataCalendarApi.json()
    // Procesamiento de datos
}

// Registra nueva cita
const appToCalendar = async (text: string) => {
    const payload = JSON.parse(text)
    await fetch('https://hook.eu2.make.com/nrbolpvmtt730jgyepvpb4qz3c0145s6', {
        method: 'POST',
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(payload)
    })
}
```

### 6. **Gestión de Estado**

#### **MemoryDB**
- **Tipo**: Almacenamiento en memoria
- **Vida útil**: Sesión de conversación
- **Limpieza**: Automática post-confirmación

#### **Estructura de Estado**
```typescript
interface BotState {
    history: History[];           // Historial de conversación
    name?: string;               // Nombre del cliente
    startDate?: string;          // Fecha de cita
    email?: string;              // Email del cliente
}
```

#### **Utilidades de Estado**
```typescript
// Almacenar en historial
const handleHistory = async (message: History, state: BotState) => {
    const history = state.get<History[]>('history') ?? []
    history.push(message)
    await state.update({ history })
}

// Obtener historial parseado
const getHistoryParse = (state: BotState, limit = 6): string => {
    const history = state.get<History[]>('history') ?? []
    const limitedHistory = history.slice(-limit)
    return limitedHistory.reduce((prev, current) => {
        const msg = current.role === 'user' 
            ? `\nCliente: "${current.content}"` 
            : `\nVendedor: "${current.content}"`
        return prev + msg
    }, '')
}
```

## 🔄 Flujo de Datos

### 1. **Recepción de Mensaje**
```
Usuario → WhatsApp → BaileysProvider → @bot-whatsapp/bot
```

### 2. **Procesamiento Inicial**
```
@bot-whatsapp/bot → Welcome Flow → Conversational Layer → Main Layer
```

### 3. **Routing Inteligente**
```
Main Layer → AI Analysis → Flow Selection → Specific Flow
```

### 4. **Ejecución de Flujo**
```
Specific Flow → AI Service → External APIs → Response Generation
```

### 5. **Respuesta al Usuario**
```
Response → @bot-whatsapp/bot → BaileysProvider → WhatsApp → Usuario
```

## 🛡️ Consideraciones de Seguridad

### **Vulnerabilidades Identificadas**
1. **API Keys en código**: OpenAI key expuesta
2. **Sin validación de entrada**: Prompts sin sanitización
3. **Endpoints públicos**: Make.com hooks sin autenticación
4. **Sin rate limiting**: Posible spam

### **Recomendaciones de Seguridad**
```typescript
// Validación de entrada
const sanitizeInput = (input: string): string => {
    return input.replace(/[<>]/g, '').trim()
}

// Rate limiting
const rateLimiter = new Map<string, number>()
const checkRateLimit = (userId: string): boolean => {
    const now = Date.now()
    const lastRequest = rateLimiter.get(userId) || 0
    if (now - lastRequest < 1000) return false // 1 segundo entre mensajes
    rateLimiter.set(userId, now)
    return true
}
```

## 📊 Métricas de Performance

### **Tiempos de Respuesta Objetivo**
- **AI Response**: <1.5 segundos
- **Calendar Query**: <2 segundos
- **Total Flow**: <3 segundos

### **Límites de Recursos**
- **Memory Usage**: <100MB por conversación
- **API Calls**: <100 requests/minuto
- **Concurrent Users**: <50 simultáneos

## 🔧 Configuración de Desarrollo

### **Variables de Entorno Requeridas**
```bash
OPEN_API_KEY=sk-...                    # OpenAI API Key
TELEGRAM_API=...                       # Telegram Bot Token (opcional)
NODE_ENV=development                   # Ambiente
```

### **Scripts de Desarrollo**
```json
{
  "scripts": {
    "dev": "npx tsx src/app.ts",
    "build": "tsc",
    "start": "node dist/app.js",
    "test": "jest"
  }
}
```

---

*Documentación técnica generada el: $(date)*
*Versión: 1.0.0*

