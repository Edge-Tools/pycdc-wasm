FROM emscripten/emsdk:5.0.7 AS builder

ENV DEBIAN_FRONTEND=noninteractive

ARG PYCDC_COMMIT=b4289760970dbc399684f1e155ec6d1ea1cc787e
ARG SOURCE_SHA256=d7d8c53d37a1a586097145b532fca02535db2f2f0999b3b1e910133c9941bf31
ARG VERSION=2026.04.07
ENV VERSION=${VERSION}

WORKDIR /src

ADD https://codeload.github.com/zrax/pycdc/tar.gz/${PYCDC_COMMIT} pycdc.tar.gz
RUN echo "${SOURCE_SHA256}  pycdc.tar.gz" | sha256sum -c - \
    && tar -xzf pycdc.tar.gz && rm pycdc.tar.gz \
    && mv pycdc-${PYCDC_COMMIT} pycdc

WORKDIR /src/pycdc

COPY js /src/pycdc/js

ENV PYCDC_LDFLAGS="\
    -lnodefs.js -lworkerfs.js \
    --post-js /src/pycdc/js/post.js \
    -fexceptions \
    -sWASM_BIGINT=1 \
    -sALLOW_MEMORY_GROWTH=1 \
    -sINITIAL_MEMORY=33554432 \
    -sMAXIMUM_MEMORY=2147483648 \
    -sSTACK_SIZE=8388608 \
    -sEXIT_RUNTIME=1 \
    -sFORCE_FILESYSTEM=1 \
    -sEXPORTED_RUNTIME_METHODS=callMain,FS,NODEFS,WORKERFS,ENV \
    -sINCOMING_MODULE_JS_API=noInitialRun,wasmBinary,locateFile,preRun,postRun,instantiateWasm,print,printErr,arguments \
    -sINVOKE_RUN=0 \
    -sMODULARIZE=1 \
    -sEXPORT_ES6=1 \
"

RUN emcmake cmake -B build \
      -DCMAKE_BUILD_TYPE=Release \
      -DCMAKE_CXX_FLAGS="-O2 -fexceptions -Wno-error" \
      -DCMAKE_EXE_LINKER_FLAGS="${PYCDC_LDFLAGS}"

RUN cmake --build build -j"$(nproc)" --target pycdc pycdas

RUN mkdir -p /dist \
    && for t in pycdc pycdas; do \
        JS="$(find build -name "$t.js" -print -quit)" ; \
        [ -n "$JS" ] || { echo "FATAL: $t.js not found in build tree"; exit 1; } ; \
        WASM="${JS%.js}.wasm" ; \
        cp "$JS" "/dist/$t.mjs" ; \
        cp "$WASM" "/dist/$t.wasm" ; \
        grep -q "export default" "/dist/$t.mjs" \
          || { echo "FATAL: $t.mjs is not an ES6 module (no 'export default')"; exit 1; } ; \
        [ "$(head -c4 "/dist/$t.wasm" | od -An -tx1 | tr -d ' \n')" = "0061736d" ] \
          || { echo "FATAL: $t.wasm has no wasm magic"; exit 1; } ; \
        [ "$(stat -c%s "/dist/$t.wasm")" -gt 65536 ] \
          || { echo "FATAL: $t.wasm is implausibly small"; exit 1; } ; \
    done \
    && ls -lh /dist

FROM scratch AS exportor
COPY --from=builder /dist /dist
