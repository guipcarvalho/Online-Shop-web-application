# ===== Build stage (legacy-friendly for Angular 7) =====
FROM node:12-bullseye AS builder
WORKDIR /app

# Copy manifests first for npm install layer caching
COPY package.json ./
# Copy lockfile if present (won't break if missing)
COPY package-lock.json* ./

# Keep npm 6 with Node 12 (better for Angular 7 trees)
RUN npm install -g npm@6

# Ensure native build toolchain is available (for any optional native deps)
RUN apt-get update && apt-get install -y --no-install-recommends python3 make g++ \
    && rm -rf /var/lib/apt/lists/*

# Install dependencies (prefer lockfile if it works; otherwise fall back)
RUN (npm ci || npm install --no-audit --no-fund)

# Pin legacy-compatible CSS toolchain to avoid browserslist/autoprefixer breaking on Node 12
RUN npm install -D postcss@7 autoprefixer@9 browserslist@4.21.9 --no-audit --no-fund

# Replace node-sass with sass (dart-sass) for better cross-platform compatibility
RUN npm uninstall node-sass --no-audit --no-fund || true
RUN npm install sass@1.32.13 --no-audit --no-fund

# Now bring in the rest of the code and build
COPY . .
RUN npm run build -- --prod

# ===== Nginx stage =====
FROM nginx:alpine
COPY nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=builder /app/dist/shop /usr/share/nginx/html
EXPOSE 80
CMD ["nginx","-g","daemon off;"]