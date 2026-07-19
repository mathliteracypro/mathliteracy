# 🚀 Production Deployment & Scaling Guide

This guide details the steps required to compile, containerize, and deploy the MathLiteracy Pro application to staging and production cloud environments.

---

## 🏛️ Deployment Strategy

MathLiteracy Pro uses a modern **multi-stage Docker build** pipeline to compile the React single-page application and deploy it behind an Express Node.js container on Google Cloud Run. This ensures minimal container cold-start times and high scalability.

---

## 🐋 Production Docker Container Config

Below is the optimized `Dockerfile` used to build production images:

```dockerfile
# ==========================================
# Stage 1: Compilation and Build
# ==========================================
FROM node:20-alpine AS builder
WORKDIR /app

# Copy dependency files
COPY package*.json ./
RUN npm ci

# Copy codebase
COPY . .

# Compile application assets
ENV NODE_ENV=production
RUN npm run build

# ==========================================
# Stage 2: Production Container Running
# ==========================================
FROM node:20-alpine AS runner
WORKDIR /app

# Ensure security checks are met (run as non-root)
RUN addgroup -g 1001 -S nodejs
RUN adduser -u 1001 -S nextjs -G nodejs

# Copy compiled files from stage 1
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/package*.json ./
COPY --from=builder /app/node_modules ./node_modules

USER nextjs
ENV NODE_ENV=production
ENV PORT=3000

EXPOSE 3000
CMD ["npm", "run", "start"]
```

---

## 🛠️ Deploying to Google Cloud Run

To build and push the container image to Google Container Registry (GCR) and deploy to Cloud Run, execute these commands:

### 1. Build and Submit Container
```bash
gcloud builds submit --tag gcr.io/mathlit-pro/applet:latest .
```

### 2. Deploy Container instance
```bash
gcloud run deploy mathlit-pro-applet \
  --image gcr.io/mathlit-pro/applet:latest \
  --platform managed \
  --region europe-west2 \
  --allow-unauthenticated \
  --port 3000 \
  --set-env-vars="NODE_ENV=production,PORT=3000"
```

---

## 🚀 Live Domain Integration (mathlit.free.nf)

The primary live domain is routed through a global CDN network to optimize performance for South African students:

1. **DNS Mapping**: A CNAME record points `mathlit.free.nf` to the Cloud Run ingress gateway.
2. **Nginx Ingress Edge Routing**:
   - SSL Certificates are renewed automatically through Let's Encrypt.
   - Nginx handles static file gzip compression and assets caching.
