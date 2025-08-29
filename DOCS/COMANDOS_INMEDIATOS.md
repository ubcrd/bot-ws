# Comandos Inmediatos - MVP Bienes Raíces

## 🚀 **Pasos Inmediatos para Comenzar**

### **1. Preparar el Entorno de Desarrollo**

```bash
# Verificar Node.js
node --version  # Debe ser 18+

# Verificar pnpm
pnpm --version

# Crear directorio del proyecto (si no existe)
mkdir bot-ws-real-estate
cd bot-ws-real-estate

# Inicializar Git (si no está inicializado)
git init
git add .
git commit -m "Initial commit: MVP Bienes Raíces setup"
```

### **2. Configurar el Proyecto Base**

```bash
# Instalar dependencias del bot framework
pnpm add @bot-whatsapp/bot@0.1.3-alpha.13
pnpm add @bot-whatsapp/provider-baileys@0.1.3-alpha.13
pnpm add @builderbot-plugins/openai-agents@^1.0.0

# Instalar dependencias adicionales para bienes raíces
pnpm add openai@^4.27.0
pnpm add date-fns@^3.3.1
pnpm add dotenv@^16.4.2
pnpm add googleapis@^128.0.0
pnpm add nodemailer@^6.9.7
pnpm add zod@^3.22.4

# Instalar dependencias de desarrollo
pnpm add -D typescript@^5.3.3
pnpm add -D @types/node@^20.10.5
pnpm add -D tsx@^4.7.1
pnpm add -D tsm@^2.3.0
pnpm add -D jest@^29.7.0
pnpm add -D @types/jest@^29.5.8
```

### **3. Crear Estructura de Carpetas**

```bash
# Crear estructura de directorios
mkdir -p src/{config,flows,services/{ai,calendar,crm,email,real-estate,integrations},types,utils}
mkdir -p tests/{flows,integrations,services}
mkdir -p docs

# Crear archivos base
touch src/app.ts
touch src/config/{database.ts,openai.ts,real-estate.ts}
touch src/flows/{index.ts,welcome.flow.ts,appointment.flow.ts,virtual-tour.flow.ts,pdf-request.flow.ts,units.flow.ts,amenities.flow.ts,general-questions.flow.ts}
touch src/services/ai/index.ts
touch src/services/calendar/index.ts
touch src/services/crm/index.ts
touch src/services/email/index.ts
touch src/services/real-estate/{project.ts,units.ts,amenities.ts,appointment.ts}
touch src/services/integrations/make.ts
touch src/types/real-estate.ts
touch src/utils/{prompts.ts,validators.ts}
touch .env.example
touch .env
touch tsconfig.json
touch package.json
```

### **4. Configurar TypeScript**

```bash
# Crear tsconfig.json
cat > tsconfig.json << 'EOF'
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "node",
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "baseUrl": "./src",
    "paths": {
      "@/*": ["*"],
      "@/config/*": ["config/*"],
      "@/flows/*": ["flows/*"],
      "@/services/*": ["services/*"],
      "@/types/*": ["types/*"],
      "@/utils/*": ["utils/*"]
    }
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "tests"]
}
EOF
```

### **5. Configurar Variables de Entorno**

```bash
# Crear .env.example
cat > .env.example << 'EOF'
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

# Proyecto Inmobiliario
PROJECT_NAME="Residencial Las Palmas"
PROJECT_EMAIL=info@residencialpalmas.com
PROJECT_PHONE=+525512345678
PROJECT_WEBSITE=https://residencialpalmas.com
EOF

# Copiar a .env
cp .env.example .env
```

### **6. Configurar package.json**

```bash
# Actualizar package.json
cat > package.json << 'EOF'
{
  "name": "bot-ws-real-estate",
  "version": "1.0.0",
  "description": "Chatbot especializado para bienes raíces",
  "type": "module",
  "scripts": {
    "dev": "tsx src/app.ts",
    "build": "tsc",
    "start": "node dist/app.js",
    "test": "jest",
    "test:watch": "jest --watch",
    "lint": "eslint src/**/*.ts",
    "format": "prettier --write src/**/*.ts"
  },
  "keywords": ["chatbot", "real-estate", "whatsapp", "ai"],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "@bot-whatsapp/bot": "0.1.3-alpha.13",
    "@bot-whatsapp/provider-baileys": "0.1.3-alpha.13",
    "@builderbot-plugins/openai-agents": "^1.0.0",
    "openai": "^4.27.0",
    "date-fns": "^3.3.1",
    "dotenv": "^16.4.2",
    "googleapis": "^128.0.0",
    "nodemailer": "^6.9.7",
    "zod": "^3.22.4"
  },
  "devDependencies": {
    "typescript": "^5.3.3",
    "@types/node": "^20.10.5",
    "tsx": "^4.7.1",
    "tsm": "^2.3.0",
    "jest": "^29.7.0",
    "@types/jest": "^29.5.8"
  }
}
EOF
```

### **7. Crear Archivo Principal**

```bash
# Crear src/app.ts
cat > src/app.ts << 'EOF'
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
EOF
```

### **8. Crear Configuración de Bienes Raíces**

```bash
# Crear src/config/real-estate.ts
cat > src/config/real-estate.ts << 'EOF'
export const realEstateConfig = {
    project: {
        name: process.env.PROJECT_NAME || "Residencial Las Palmas",
        description: "Proyecto residencial de lujo en el corazón de la ciudad",
        developer: "Constructora ABC",
        location: {
            address: "Av. Principal 123",
            city: "Ciudad de México",
            state: "CDMX"
        },
        contact: {
            phone: process.env.PROJECT_PHONE || "+52 55 1234 5678",
            email: process.env.PROJECT_EMAIL || "info@residencialpalmas.com",
            website: process.env.PROJECT_WEBSITE || "https://residencialpalmas.com"
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
        },
        {
            id: "3br",
            name: "Departamento 3BR", 
            size: { min: 140, max: 160 },
            priceRange: { min: 3000000, max: 4000000 }
        },
        {
            id: "penthouse",
            name: "Penthouse",
            size: { min: 200, max: 250 },
            priceRange: { min: 4000000, max: 5000000 }
        }
    ],
    amenities: [
        {
            id: "pool",
            name: "Alberca",
            category: "Recreación",
            icon: "🏊‍♂️",
            description: "Alberca infinita con vista panorámica"
        },
        {
            id: "gym",
            name: "Gimnasio",
            category: "Recreación", 
            icon: "🏋️‍♂️",
            description: "Gimnasio equipado 24/7"
        },
        {
            id: "tennis",
            name: "Cancha de Tenis",
            category: "Recreación",
            icon: "🎾", 
            description: "Cancha de tenis profesional"
        }
    ]
}
EOF
```

### **9. Crear Tipos TypeScript**

```bash
# Crear src/types/real-estate.ts
cat > src/types/real-estate.ts << 'EOF'
export interface RealEstateProject {
    name: string;
    description: string;
    developer: string;
    location: {
        address: string;
        city: string;
        state: string;
        coordinates?: {
            lat: number;
            lng: number;
        };
    };
    propertyTypes: PropertyType[];
    priceRange: {
        min: number;
        max: number;
        currency: string;
    };
    features: {
        totalUnits: number;
        availableUnits: number;
        constructionStatus: string;
        deliveryDate: string;
    };
    amenities: Amenity[];
    contact: {
        phone: string;
        email: string;
        website: string;
        officeHours: string;
    };
    digitalAssets: {
        virtualTours: VirtualTour[];
        brochures: Brochure[];
        floorPlans: FloorPlan[];
        images: Image[];
    };
}

export interface PropertyType {
    id: string;
    name: string;
    description: string;
    size: {
        min: number;
        max: number;
    };
    bedrooms: number;
    bathrooms: number;
    parking: number;
    priceRange: {
        min: number;
        max: number;
    };
    availableUnits: number;
    floorPlans: FloorPlan[];
}

export interface Amenity {
    id: string;
    name: string;
    category: string;
    description: string;
    icon: string;
    status: "available" | "construction" | "planned";
    images: string[];
}

export interface VirtualTour {
    id: string;
    name: string;
    type: "360" | "video" | "drone";
    url: string;
    thumbnail: string;
    duration: number;
    description: string;
}

export interface Brochure {
    id: string;
    name: string;
    type: "general" | "specific" | "financing";
    url: string;
    size: number;
    pages: number;
    description: string;
}

export interface FloorPlan {
    id: string;
    name: string;
    url: string;
    size: number;
    description: string;
}

export interface Image {
    id: string;
    name: string;
    url: string;
    alt: string;
    category: string;
}
EOF
```

### **10. Instalar y Verificar**

```bash
# Instalar dependencias
pnpm install

# Verificar que todo funciona
pnpm dev

# Si hay errores, verificar:
# 1. Variables de entorno configuradas
# 2. Node.js versión correcta
# 3. Dependencias instaladas
```

---

## 🎯 **Próximos Pasos Inmediatos**

### **Después de la configuración inicial:**

1. **Configurar credenciales reales** en `.env`
2. **Implementar el primer flujo** (bienvenida)
3. **Probar con WhatsApp** en modo desarrollo
4. **Implementar flujo de agendar cita**
5. **Configurar Make.com** webhooks

### **Comandos para desarrollo:**

```bash
# Ejecutar en modo desarrollo
pnpm dev

# Ejecutar tests
pnpm test

# Construir para producción
pnpm build

# Ejecutar en producción
pnpm start
```

---

## 🚨 **Solución de Problemas Comunes**

### **Error: Module not found**
```bash
# Reinstalar dependencias
rm -rf node_modules pnpm-lock.yaml
pnpm install
```

### **Error: OpenAI API key**
```bash
# Verificar variable de entorno
echo $OPENAI_API_KEY
# O agregar al .env
echo "OPENAI_API_KEY=tu_api_key_aqui" >> .env
```

### **Error: TypeScript compilation**
```bash
# Verificar configuración
npx tsc --noEmit
# O reinstalar TypeScript
pnpm add -D typescript@latest
```

---

*Comandos Inmediatos generado el: $(date)*
*Versión: 1.0.0*
