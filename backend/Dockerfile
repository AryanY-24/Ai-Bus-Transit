# Use official lightweight Python 3.13 image
FROM python:3.13-slim

# Set environment variables
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

WORKDIR /app

# Install system dependencies if required
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

# -------------------------------------------------
# 2️⃣ Copy and install python dependencies
# -------------------------------------------------
COPY backend/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# -------------------------------------------------
# 3️⃣ Copy backend app and trained ML models
# -------------------------------------------------
COPY backend/app/    ./app/
COPY backend/models/ ./models/

# -------------------------------------------------
# 4️⃣ Expose FastAPI default port
# -------------------------------------------------
EXPOSE 8000

# -------------------------------------------------
# 5️⃣ Run with uvicorn in production mode
# -------------------------------------------------
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]