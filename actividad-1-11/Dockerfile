# ==============================================================================
# ETAPA 1: Builder (Compilación / Instalación de dependencias)
# ==============================================================================
# Uso de imagen oficial con tag específico e inmutable (evitando :latest)
FROM python:3.12.7-slim-bookworm AS builder

# Configuración para evitar escrituras innecesarias en disco y buffereado de logs
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

WORKDIR /install

# Optimización de capas: copiamos primero solo el manifiesto de dependencias
COPY requirements.txt .

# Instalación en el prefijo /install sin utilizar la caché de pip para reducir peso
RUN pip install --no-cache-dir --prefix=/install -r requirements.txt

# ==============================================================================
# ETAPA 2: Runtime (Entorno final mínimo y seguro)
# ==============================================================================
FROM python:3.12.7-slim-bookworm AS runner

# Principio DevSecOps: Mínimo privilegio
# Creamos un grupo y usuario de sistema específico sin privilegios de root (UID/GID 10001)
RUN groupadd -g 10001 appgroup && \
    useradd -u 10001 -g appgroup -s /sbin/nologin -d /app appuser

WORKDIR /app

# Copiamos únicamente los paquetes de Python instalados desde la etapa builder
COPY --from=builder /install /usr/local

# Copiamos el código fuente de la aplicación con la propiedad del usuario no-root
COPY --chown=appuser:appgroup main.py .

# Exponemos documentalmente el puerto definido
EXPOSE 8080

# Cambiamos al usuario sin privilegios
USER appuser

# Healthcheck para validar el estado del contenedor periódicamente
HEALTHCHECK --interval=30s --timeout=5s --start-period=5s --retries=3 \
  CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8080/health').read()" || exit 1

# Sintaxis de ejecución segura: Exec form (array) utilizando servidor de producción WSGI
CMD ["gunicorn", "--bind", "0.0.0.0:8080", "--workers", "2", "--threads", "2", "main:app"]