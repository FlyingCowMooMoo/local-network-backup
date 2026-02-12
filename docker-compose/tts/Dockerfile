FROM python:3.12-slim AS build

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        build-essential \
        cmake \
        ninja-build \
        git && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /build
RUN git clone --depth 1 https://github.com/OHF-Voice/piper1-gpl.git . && \
    pip install --no-cache-dir .[http]

# ---------------------------------------------------------------------------
FROM python:3.12-slim

RUN apt-get update && \
    apt-get install -y --no-install-recommends curl && \
    rm -rf /var/lib/apt/lists/*

COPY --from=build /usr/local/lib/python3.12/site-packages /usr/local/lib/python3.12/site-packages
COPY --from=build /usr/local/bin /usr/local/bin

# ---------------------------------------------------------------------------
# Download German voices from HuggingFace
# ---------------------------------------------------------------------------
ARG VOICE_URL=https://huggingface.co/rhasspy/piper-voices/resolve/main

RUN mkdir -p /data && \
    for voice in \
        "de/de_DE/thorsten/medium/de_DE-thorsten-medium" \
        "de/de_DE/thorsten/high/de_DE-thorsten-high" \
        "de/de_DE/thorsten/low/de_DE-thorsten-low" \
        "de/de_DE/thorsten_emotional/medium/de_DE-thorsten_emotional-medium" \
        "de/de_DE/kerstin/low/de_DE-kerstin-low" \
        "de/de_DE/karlsson/low/de_DE-karlsson-low" \
        "de/de_DE/eva_k/x_low/de_DE-eva_k-x_low" \
        "de/de_DE/ramona/low/de_DE-ramona-low" \
        "de/de_DE/pavoque/low/de_DE-pavoque-low" \
    ; do \
        name="${voice##*/}"; \
        curl -fSL "${VOICE_URL}/${voice}.onnx?download=true"      -o "/data/${name}.onnx"; \
        curl -fSL "${VOICE_URL}/${voice}.onnx.json?download=true"  -o "/data/${name}.onnx.json"; \
    done

# Remove curl – no longer needed at runtime
RUN apt-get purge -y curl && apt-get autoremove -y && rm -rf /var/lib/apt/lists/*

EXPOSE 5000

ENTRYPOINT ["python3", "-m", "piper.http_server", "--host", "0.0.0.0", "--data-dir", "/data"]
CMD ["--model", "/data/de_DE-thorsten-medium.onnx"]
