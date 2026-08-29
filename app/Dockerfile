# =========================
# Stage 1: Builder
# =========================
FROM python:3.12-slim AS builder

WORKDIR /app

# Prevent Python from writing .pyc files and enable unbuffered logs
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

# Create an isolated virtual environment
RUN python -m venv /opt/venv

# Make the virtual environment the default Python environment
ENV PATH="/opt/venv/bin:$PATH"

# Copy only dependency files first for better Docker layer caching
COPY requirements.txt .

# Install dependencies into the virtual environment
RUN pip install --no-cache-dir --upgrade pip \
    && pip install --no-cache-dir -r requirements.txt


# =========================
# Stage 2: Runtime
# =========================
FROM python:3.12-slim AS runtime

WORKDIR /app

# Python runtime settings
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PATH="/opt/venv/bin:$PATH"

# Create a non-root user
RUN groupadd --system app \
    && useradd --system --gid app --create-home app

# Copy only the virtual environment from the builder
COPY --from=builder /opt/venv /opt/venv

# Copy application source
COPY --chown=app:app . .

# Run as non-root user
USER app

EXPOSE 5000

CMD ["gunicorn", "--bind", "0.0.0.0:5000", "--workers", "2", "app:app"]
