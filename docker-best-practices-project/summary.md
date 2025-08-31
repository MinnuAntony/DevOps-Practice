
# Dockerfile for front end

```dockerfile
FROM node:20.11-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci --no-audit --no-fund
COPY . .
RUN npm run build && npm cache clean --force

FROM nginx:1.25.3-alpine
COPY nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=build /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### **1. Multi-stage build**

```dockerfile
FROM node:20.11-alpine AS build
...
FROM nginx:1.25.3-alpine
```

* **Why important:** Only the final compiled files (`dist`) go into the runtime image. Node and build tools don’t bloat the final image.
* **Impact:** Much smaller image and faster deployment.

---

### **2. Copy `package*.json` separately and run `npm ci`**

```dockerfile
COPY package*.json ./
RUN npm ci --no-audit --no-fund
```

* **Why important:** Docker caches this layer. If your code changes but dependencies don’t, Docker **reuses the cached dependencies**, saving time.
* **Impact:** Faster builds, efficient caching.

---

### **3. Clean npm cache after build**

```dockerfile
RUN npm run build && npm cache clean --force
```

* **Why important:** Removes temporary cache files created during `npm install`.
* **Impact:** Smaller image size.

---

### **4. Copy only the build artifacts to Nginx**

```dockerfile
COPY --from=build /app/dist /usr/share/nginx/html
```

* **Why important:** Only the compiled static files are needed in production.
* **Impact:** Keeps runtime image minimal and secure.

---

### **5. Use Alpine images**

```dockerfile
FROM node:20.11-alpine
FROM nginx:1.25.3-alpine
```

* **Why important:** Alpine is very small (\~5–10MB) and secure.
* **Impact:** Lightweight, faster to pull and deploy.

---

# Dockerfile for user-service - Python FastAPI

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt \
    && python -m compileall . \
    && addgroup --system appgroup \
    && adduser --system --ingroup appgroup appuser \
    && chown -R appuser:appgroup /app
EXPOSE 5000
COPY . .
USER appuser
CMD ["python", "-m", "app.main"]
````

## **Best Practices**

1. **Using slim base image** → lightweight and faster to pull.
2. **Copying `requirements.txt` first** → caches dependency installation; changes in source code won't reinstall packages unnecessarily.
3. **Compiling Python files** → improves runtime performance.
4. **Creating non-root user (`appuser`)** → security best practice.
5. **Copying source code in the last layer** → **improves cache efficiency**: only changes to your app code trigger rebuilds of this layer, not dependency installation.
6. **`--no-cache-dir` in pip** → keeps image smaller by not storing pip cache.

---

# Dockerfile for Go Microservice (Multi-stage Build)

```dockerfile
FROM golang:1.21-alpine AS builder
RUN apk add --no-cache git 
WORKDIR /app
COPY . .
RUN go mod tidy
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-s -w" -o expense-service main.go

FROM scratch
COPY --from=builder /app/expense-service /expense-service
USER 1000:1000
EXPOSE 8080
CMD ["/expense-service"]
````

### **Best Practices in Build Stage**

1. **Multi-stage build** → separates compilation from runtime, keeping the final image minimal.
2. **Alpine base image** → small and secure, reducing build image size.
3. **Copying source code** before `go mod tidy` → all dependencies are installed correctly.
4. **Statically linked binary (`CGO_ENABLED=0`)** → ensures binary can run in minimal `scratch` image.

### **Best Practices in Run Stage**

1. **Using `scratch`** → zero-size base image; only your binary exists, making the runtime extremely lightweight.
2. **Copying only the binary** → no source code, no dependencies, keeping the final image minimal.
3. **Non-root user** → improves security.
4. **Minimal layers** → each `COPY` and `RUN` creates a layer; only essential steps are included.

---

