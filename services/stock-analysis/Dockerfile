# Stock Analysis Service Dockerfile
FROM python:3.10-slim-bookworm

ARG TARGETARCH

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PIP_NO_CACHE_DIR=1 \
    TZ=Asia/Shanghai

WORKDIR /app

# Install system dependencies
RUN mkdir -p /app/logs /app/data /app/config && \
    sed -i 's|deb.debian.org|mirrors.aliyun.com|g' /etc/apt/sources.list.d/debian.sources || true && \
    echo 'Acquire::Retries "3";' > /etc/apt/apt.conf.d/80-retries && \
    echo 'Acquire::http::Timeout "30";' >> /etc/apt/apt.conf.d/80-retries && \
    echo 'Acquire::https::Timeout "30";' >> /etc/apt/apt.conf.d/80-retries && \
    (apt-get update || apt-get update || apt-get update) && \
    apt-get install -y --no-install-recommends --fix-missing \
        ca-certificates curl fontconfig fonts-noto-cjk wget xvfb && \
    if [ "$TARGETARCH" = "arm64" ]; then \
        PANDOC_ARCH="arm64"; WKHTMLTOPDF_ARCH="arm64"; \
    else \
        PANDOC_ARCH="amd64"; WKHTMLTOPDF_ARCH="amd64"; \
    fi && \
    wget -q https://github.com/jgm/pandoc/releases/download/3.8.2.1/pandoc-3.8.2.1-1-${PANDOC_ARCH}.deb && \
    dpkg -i pandoc-3.8.2.1-1-${PANDOC_ARCH}.deb && \
    rm pandoc-3.8.2.1-1-${PANDOC_ARCH}.deb && \
    wget -q https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6.1-3/wkhtmltox_0.12.6.1-3.bookworm_${WKHTMLTOPDF_ARCH}.deb && \
    apt-get install -y --no-install-recommends ./wkhtmltox_0.12.6.1-3.bookworm_${WKHTMLTOPDF_ARCH}.deb && \
    rm wkhtmltox_0.12.6.1-3.bookworm_${WKHTMLTOPDF_ARCH}.deb && \
    fc-cache -fv && \
    rm -rf /var/lib/apt/lists/*

# Install Python dependencies
COPY pyproject.toml requirements.txt ./
RUN pip install --upgrade pip -i https://pypi.tuna.tsinghua.edu.cn/simple && \
    pip install --prefer-binary . -i https://pypi.tuna.tsinghua.edu.cn/simple && \
    pip install --prefer-binary pdfkit -i https://pypi.tuna.tsinghua.edu.cn/simple

# Copy source code
COPY src/app ./app
COPY src/tradingagents ./tradingagents
COPY config ./config
COPY scripts ./scripts
COPY .env.example ./.env

EXPOSE 8000
ENV DOCKER_CONTAINER=true

CMD ["python", "-m", "uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
