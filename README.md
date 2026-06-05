# x402 Starter Kit - Guía Completa

HTTP 402 payment integration with Thirdweb on Avalanche Fuji testnet + z.ai free AI models.

**Costo total: $0** - Todo es testnet y el modelo de IA es gratuito (GLM-4.5-Flash).

---

## ¿Qué es este proyecto?

Una aplicación web que implementa **micropagos en criptomonedas** usando el protocolo HTTP 402 para:
- Cobrar por servicios API ($0.01 - $0.15 USDC)
- Chat con IA (pago por tokens usados)
- Agentes autónomos con presupuesto pre-autorizado

---

## Inicio Rápido (GitHub Codespaces)

### Paso 1: Abrir en Codespaces

1. Haz clic en **"Code"** → **"Codespaces"** → **"Create codespace on master"**
2. Espera 1-2 minutos mientras se instala todo automáticamente
3. El archivo `.env.local` se creará automáticamente

### Paso 2: Configurar Thirdweb

#### 2.1 Crear cuenta y proyecto

1. Ve a https://thirdweb.com/dashboard
2. Conecta tu wallet (MetaMask, WalletConnect, etc.)
3. Crea un nuevo proyecto o selecciona uno existente

#### 2.2 Obtener credenciales

1. En tu proyecto, ve a **Settings**
2. Copia el **Client ID** → guardar para `.env.local`
3. Copia el **Secret Key** → guardar para `.env.local`
4. **IMPORTANTE**: Desactiva la restricción de dominio del Client ID
   - Cada Codespace tiene una URL única (*.app.github.dev)
   - Sin esto, la conexión de wallet fallará

#### 2.3 Crear Smart Account (Facilitador)

El facilitador paga el gas de todas las transacciones para que sea "gasless" para los usuarios.

1. En el dashboard, ve a **"Server Wallets"**
2. Activa el switch **"Show ERC4337 Smart Account"**
3. Selecciona la red **"Avalanche Fuji Testnet"**
4. Copia la dirección generada → `THIRDWEB_SERVER_WALLET_ADDRESS`

**IMPORTANTE**: Solo ERC4337 Smart Accounts son soportados. NO uses ERC-7702.

#### 2.4 Fondear el Smart Account con AVAX

1. Copia la dirección del Smart Account
2. Ve a un faucet de Avalanche Fuji:
   - https://core.app/tools/testnet-faucet/
   - https://faucet.avax.network/
3. Solicita AVAX de testnet
4. Espera 1-2 minutos a que lleguen los fondos

### Paso 3: Obtener API Key de z.ai

1. Ve a https://z.ai/manage-apikey/apikey-list
2. Crea cuenta o inicia sesión
3. Genera una API Key
4. Cópiala → `ZAI_API_KEY`

**Modelo gratuito**: `glm-4.5-flash` (sin costo, sin límites de llamadas)

### Paso 4: Configurar variables de entorno

Abre `.env.local` en la raíz del proyecto y completa:

```bash
# Thirdweb credentials
NEXT_PUBLIC_THIRDWEB_CLIENT_ID=tu_client_id_aqui
THIRDWEB_SECRET_KEY=tu_secret_key_aqui

# Smart Account (facilitador que paga el gas)
THIRDWEB_SERVER_WALLET_ADDRESS=0xTuSmartAccountAqui

# Wallet que recibe los pagos
MERCHANT_WALLET_ADDRESS=0xTuWalletPersonalAqui

# z.ai API Key (modelo gratuito GLM-4.5-Flash)
ZAI_API_KEY=tu_api_key_de_zai_aqui
```

### Paso 5: Levantar la aplicación

```bash
npm run dev
```

Abre el puerto 3000 en una **pestaña normal del navegador** (no en el preview de VS Code).

### Paso 6: Conectar wallet y obtener USDC

#### Conectar wallet

**Opción A: Con extensión (MetaMask, etc.)**
- Haz clic en "Connect Wallet" y selecciona tu extensión

**Opción B: Sin extensión (In-App Wallet)**
- Haz clic en "Connect Wallet" → Email o Google
- Thirdweb crea un wallet embebido (no necesitas instalar nada)

#### Obtener USDC de testnet

Los pagos se hacen en USDC, no en AVAX.

1. Asegúrate de estar en **Avalanche Fuji Testnet**
2. Copia tu dirección de wallet
3. Ve a https://faucet.circle.com
4. Selecciona red **"Avalanche Fuji"**
5. Solicita USDC
6. Espera 1-2 minutos

**Contrato USDC en Fuji**: `0x5425890298aed601595a70AB815c96711a31Bc65`

---

## Probar la aplicación

### 1. Basic Endpoint ($0.01 USDC)
- Click en "Test Basic"
- Autoriza el pago en tu wallet
- Recibe respuesta del servidor

### 2. Premium Endpoint ($0.15 USDC)
- Click en "Test Premium"
- Autoriza el pago
- Recibe respuesta

### 3. AI Chat (pago por tokens)
- Escribe un mensaje
- Autoriza hasta $0.50 (se cobra solo lo usado)
- El costo se calcula por tokens: $0.001 por 1K tokens
- Respuesta generada por GLM-4.5-Flash (gratis)

### 4. AI Agent (presupuesto pre-autorizado)
- Autoriza un presupuesto (ej: $0.75)
- El agente hace múltiples llamadas a $0.02 cada una
- El presupuesto expira en 1 hora

---

## Flujo de pagos

```
1. Usuario autoriza → Firma en su wallet
2. Backend verifica → Confirma la autorización
3. Servicio se ejecuta → Llama a la IA o procesa
4. Se calcula el costo → Basado en tokens o fijo
5. Se transfiere USDC → De usuario a MERCHANT_WALLET_ADDRESS
6. Usuario recibe respuesta
```

**¿Quién paga el gas?** El Smart Account (facilitador) paga todo el gas. Los usuarios solo firman autorizaciones y pagan en USDC. Esto se llama **gasless transactions** o **sponsored transactions**.

---

## Instalación local (alternativa a Codespaces)

```bash
git clone https://github.com/carlos-israelj/x402-starter-kit.git
cd x402-starter-kit
npm install
cp .env.example .env.local
# Editar .env.local con tus credenciales
npm run dev
```

Requisitos:
- Node.js 20.9+ (Next.js 16 requirement)
- npm o yarn

---

## Arquitectura del proyecto

```
┌─────────────┐
│   Usuario   │
└──────┬──────┘
       │ Conecta wallet + Autoriza pago
       ▼
┌─────────────────────┐
│  Frontend (Next.js) │
│  - Thirdweb SDK v5  │
│  - shadcn/ui        │
└──────┬──────────────┘
       │ Request + firma
       ▼
┌─────────────────────┐
│   Backend (API)     │
│  - Verifica pago    │
│  - Llama a z.ai     │
│  - Calcula costo    │
└──────┬──────────────┘
       │
       ├─────────────────┐
       ▼                 ▼
┌──────────────┐   ┌────────────────┐
│ z.ai (IA)    │   │ Avalanche Fuji │
│ GLM-4.5-Flash│   │ USDC testnet   │
└──────────────┘   └────────────────┘
```

---

## Stack técnico

- **Next.js 16** (App Router, TypeScript)
- **Thirdweb SDK v5** (Connect wallets, ERC4337)
- **Tailwind CSS** + **shadcn/ui**
- **z.ai** (GLM-4.5-Flash - modelo gratuito)
- **Avalanche Fuji** (testnet para USDC)

---

## Estructura de archivos

```
x402-starter-kit/
├── app/
│   ├── api/
│   │   ├── basic/route.ts          # Endpoint básico ($0.01)
│   │   ├── premium/route.ts        # Endpoint premium ($0.15)
│   │   ├── ai-chat/route.ts        # Chat con IA (pago por tokens)
│   │   └── agent/route.ts          # Agente autónomo
│   ├── layout.tsx                   # Layout principal
│   └── page.tsx                     # Página principal
├── components/
│   ├── ai-agent-mode.tsx           # UI del agente autónomo
│   ├── human-payment-mode.tsx      # UI de pagos manuales
│   └── ui/                          # Componentes shadcn
├── lib/
│   ├── constants.ts                 # Configuración (z.ai, precios, etc.)
│   ├── thirdweb.ts                  # Cliente de Thirdweb
│   └── token-pricing.ts             # Cálculo de costos por token
├── .devcontainer/
│   └── devcontainer.json            # Configuración de Codespaces
├── .env.example                     # Plantilla de variables
└── .env.local                       # Variables de entorno (NO commitear)
```

---

## Troubleshooting

### Error: "Insufficient funds"
- **Causa**: No tienes suficiente USDC en tu wallet
- **Solución**: Ve al faucet de Circle y solicita más USDC

### Error: "Transaction failed" o "Out of gas"
- **Causa**: El Smart Account no tiene AVAX
- **Solución**: Fondea el Smart Account con AVAX del faucet de Fuji

### La wallet no se conecta
- **Causa**: Estás usando el preview de VS Code
- **Solución**: Abre la URL en una pestaña normal del navegador

### Error: "Invalid API key" en AI Chat
- **Causa**: La API key de z.ai no es válida
- **Solución**: Verifica `ZAI_API_KEY` en `.env.local`

### Error: "Client ID restricted"
- **Causa**: El Client ID de Thirdweb tiene restricción de dominio
- **Solución**: En el dashboard de Thirdweb, desactiva la restricción

---

## Features avanzados

### Payment Modes

**Human Payment Mode**
- Pagos manuales autorizados por el usuario
- Montos fijos o basados en uso real
- Logging de transacciones en tiempo real

**AI Agent Mode**
- Presupuesto pre-autorizado
- Agente autónomo que paga servicios
- Expiración de autorización (1 hora por defecto)

### Pricing Models

1. **Fixed Price**: Endpoints básico y premium (montos fijos)
2. **Token-Based**: AI Chat (pago por tokens usados)
3. **Budget Authorization**: AI Agent (presupuesto con expiración)

### Security Features

- Verificación de firma en backend
- Normalización de firmas para Avalanche
- Rate limiting por cliente
- Expiración de autorizaciones

---

## Recursos

- **Thirdweb Docs**: https://portal.thirdweb.com/
- **z.ai**: https://z.ai/
- **Avalanche Docs**: https://docs.avax.network/
- **Fuji Explorer**: https://testnet.snowtrace.io/
- **Circle USDC Faucet**: https://faucet.circle.com
- **HTTP 402 Spec**: https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/402

---

## Build y Deploy

### Build local

```bash
npm run build
npm start
```

### Deploy en Vercel

1. Push tu fork a GitHub
2. Importa el proyecto en Vercel
3. Configura las variables de entorno en Vercel
4. Deploy automático

**Nota**: Recuerda actualizar las restricciones de dominio en Thirdweb para incluir tu dominio de producción.

---

## Checklist de configuración

- [ ] Codespace creado o proyecto clonado localmente
- [ ] Thirdweb Client ID obtenido (sin restricción de dominio)
- [ ] Thirdweb Secret Key obtenido
- [ ] Smart Account ERC4337 creado en Fuji
- [ ] Smart Account fondeado con AVAX
- [ ] Merchant wallet definida
- [ ] z.ai API Key obtenida
- [ ] `.env.local` completo y guardado
- [ ] `npm run dev` ejecutándose sin errores
- [ ] Wallet conectada en el navegador
- [ ] Wallet fondeada con USDC de testnet
- [ ] Primera transacción exitosa

---

## Contribuir

¿Encontraste un bug o tienes una mejora?

1. Fork el proyecto
2. Crea una rama (`git checkout -b feature/mejora`)
3. Commit tus cambios (`git commit -m 'Add: nueva funcionalidad'`)
4. Push a la rama (`git push origin feature/mejora`)
5. Abre un Pull Request

---

## License

MIT

---

¡Ahora tienes un sistema de micropagos web3 funcionando! 🎉
