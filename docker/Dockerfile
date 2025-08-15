# ---- Base Stage ----
FROM python:3.10-slim-bookworm as base
ENV PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    # Poetry settings
    POETRY_NO_INTERACTION=1 \
    POETRY_VIRTUALENVS_IN_PROJECT=1 \
    POETRY_VIRTUALENVS_CREATE=1

WORKDIR /app

# Install system dependencies, including procps for the healthcheck
RUN apt-get update && apt-get install -y --no-install-recommends procps && rm -rf /var/lib/apt/lists/* # <-- UPDATED THIS SECTION

# Install poetry
RUN pip install --upgrade pip
RUN pip install poetry

# ---- Build Stage ----
FROM base as builder

# Copy only the files needed for dependency installation
COPY pyproject.toml poetry.lock ./

# Install dependencies ONLY, skipping the main project installation
RUN poetry install --without dev --no-root

# Copy the rest of the application code
COPY . .

# Install the main application package itself, creating the 'trading_bot' command
RUN poetry install --without dev

# ---- Final Stage ----
FROM base as final

# Copy the virtual environment from the build stage
COPY --from=builder /app/.venv/ .venv/
# Copy the application code from the builder stage
COPY --from=builder /app/tradingbot/ ./tradingbot/
COPY --from=builder /app/config/ ./config/

# Activate the virtual environment
ENV PATH="/app/.venv/bin:$PATH"

HEALTHCHECK --interval=1m --timeout=30s --start-period=5m --retries=3 CMD pgrep -f trading_bot || exit 1

CMD ["trading_bot"]