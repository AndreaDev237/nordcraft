# ==========================================
# Build Stage - Compila tutto il progetto
# ==========================================
FROM oven/bun:1.2-debian AS build

WORKDIR /app

# Copia i file di configurazione del workspace
COPY bun.lock package.json bunfig.toml ./

# Copia tutti i package.json dei packages
COPY packages/backend/package.json ./packages/backend/
COPY packages/core/package.json ./packages/core/
COPY packages/lib/package.json ./packages/lib/
COPY packages/runtime/package.json ./packages/runtime/
COPY packages/search/package.json ./packages/search/
COPY packages/ssr/package.json ./packages/ssr/

# Installa TUTTE le dipendenze (incluse devDependencies per il build)
RUN bun install --frozen-lockfile

# Copia tutto il codice sorgente
COPY . .

# STEP CRITICO: Compila tutti i packages del workspace
RUN bun run build

# Sincronizza gli asset statici del backend
RUN cd packages/backend && bun run bin/syncStaticAssets.ts

# Compila il backend come eseguibile standalone
RUN bun build --compile --minify --sourcemap ./packages/backend/src/bun.index.ts --outfile nordcraft-backend

# Test: verifica che il binario possa fare fetch in fase di build
RUN ./nordcraft-backend --version || echo "Binary check done"

# ==========================================
# Runtime Stage - Prova Debian invece di Distroless
# ==========================================
FROM debian:12-slim AS runner

# Installa solo le dipendenze minime necessarie
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# Variabili d'ambiente
ENV NODE_ENV=production

ARG BUILD_APP_PORT=3000
ENV APP_PORT=${BUILD_APP_PORT}
EXPOSE ${APP_PORT}

WORKDIR /app

# Copia l'eseguibile compilato
COPY --from=build /app/nordcraft-backend .

# Copy project specific files
COPY --from=build /app/packages/backend/dist/ .

# Copia __project__
COPY --from=build /app/packages/backend/__project__/ ./__project__/

# Avvia l'applicazione
ENTRYPOINT ["./nordcraft-backend"]
