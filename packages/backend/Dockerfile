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
# Questo crea le directory dist/ per core, runtime, ssr, lib, search
RUN bun run build

# Sincronizza gli asset statici del backend
# (ora funziona perché packages/runtime/dist/ esiste)
RUN cd packages/backend && bun run bin/syncStaticAssets.ts

# Compila il backend come eseguibile standalone
RUN bun build --compile --minify --sourcemap ./packages/backend/src/bun.index.ts --outfile nordcraft-backend

# ==========================================
# Runtime Stage - Immagine finale minimal
# ==========================================
FROM gcr.io/distroless/base-debian12 AS runner

# Variabili d'ambiente
ENV NODE_ENV=production

ARG BUILD_APP_PORT=3000
ENV APP_PORT=${BUILD_APP_PORT}
EXPOSE ${APP_PORT}

WORKDIR /app

# Copia l'eseguibile compilato
COPY --from=build /app/nordcraft-backend .

# Copia la directory dist con gli asset statici
COPY --from=build /app/packages/backend/dist/ ./dist/

# Avvia l'applicazione
ENTRYPOINT ["./nordcraft-backend"]
