FROM xoelabs/dockery-odoo-base:11.0
# ============================================================
# Convention about required libraries
# ============================================================

USER root

RUN pip --quiet --quiet install --no-cache-dir redis prometheus_client; \
    find /usr/local -depth \
        \( \
            \( -type d -a \( -name test -o -name tests \) \) \
            -o \
            \( -type f -a \( -name '*.pyc' -o -name '*.pyo' \) \) \
        \) -exec rm -rf '{}' +;

# ============================================================
# Convention about environment variables
# ============================================================

ENV ODOO_BASEPATH        "/opt/odoo"
ENV ODOO_RC              "${ODOO_BASEPATH}/cfg.d"
ENV ODOO_MIG             "${ODOO_BASEPATH}/migration.yaml"
ENV ODOO_MIG_DIR         "${ODOO_BASEPATH}/migration.d"
ENV ODOO_CMD             "${ODOO_BASEPATH}/vendor/odoo/cc/odoo-bin"
ENV ODOO_FRM             "${ODOO_BASEPATH}/vendor/odoo/cc"
ENV ODOO_VENDOR          "${ODOO_BASEPATH}/vendor"
ENV ODOO_SRC             "${ODOO_BASEPATH}/src"
ENV PATCHES_DIR          "${ODOO_BASEPATH}/patches.d"

# ============================================================
# Ship with conventional files (patches, entrypoints, bin)
# ============================================================

COPY patches.d "${PATCHES_DIR}"
COPY entrypoint.d/* /entrypoint.d/
COPY bin/* /usr/local/bin/
RUN /bin/bash -c 'shopt -s dotglob \
 && chmod +x /entrypoint.d/* \
 && chmod +x /usr/local/bin/* \
 && shopt -u dotglob'

# ============================================================
# Forward enforce minimal naming scheme on secondary build
# ============================================================

ONBUILD ENV LAYER_SUFFIX "-9"
ONBUILD ENV ODOO_VENDOR  "${ODOO_VENDOR} ${ODOO_BASEPATH}/vendor${LAYER_SUFFIX}"
ONBUILD ENV ODOO_SRC     "${ODOO_SRC} ${ODOO_BASEPATH}/src${LAYER_SUFFIX}"
ONBUILD ENV ODOO_CMD     "${ODOO_BASEPATH}/vendor${LAYER_SUFFIX}/odoo/cc/odoo-bin"
ONBUILD ENV ODOO_FRM     "${ODOO_BASEPATH}/vendor${LAYER_SUFFIX}/odoo/cc"
ONBUILD COPY --chown=odoo:odoo  migration.yaml              "${ODOO_MIG}"
ONBUILD COPY --chown=odoo:odoo  migration.d                 "${ODOO_MIG_DIR}"
ONBUILD COPY --chown=odoo:odoo  patches.d/*                 "${PATCHES_DIR}/"
ONBUILD COPY --chown=odoo:odoo  vendor                      "${ODOO_BASEPATH}/vendor${LAYER_SUFFIX}"
ONBUILD COPY --chown=odoo:odoo  src                         "${ODOO_BASEPATH}/src${LAYER_SUFFIX}"
# ============================================================

WORKDIR ${ODOO_BASEPATH}/src


USER root

# Some helpeful tools
# Size: ca. 57MB
RUN apt-get -qq update && apt-get -qq install -y curl wget mtr net-tools gnupg

# Local browser testing
# Size: ca. 291MB
RUN wget --quiet -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add - \
    && echo 'deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main' | tee /etc/apt/sources.list.d/google-chrome.list \
    && apt-get -qq update && apt-get -qq install -y --no-install-recommends google-chrome-stable > /dev/null

# Environement
ENV WDB_NO_BROWSER_AUTO_OPEN=True \
    WDB_SOCKET_SERVER=wdb \
    WDB_WEB_PORT=1984 \
    WDB_WEB_SERVER=localhost \
    PYTHONPATH=${PYTHONPATH}:${ODOO_FRM} \
    ODOO_TEMPLATES_DIR="/templates" \
    DODOO_LOAD_OUT=/var/lib/dodoo-loader/logs.json

# Python packages
# Size: ca. 138MB
RUN set -ex; \
    \
    savedAptMark="$(apt-mark showmanual)"; \
    apt-get update; \
    apt-get install -y --no-install-recommends build-essential automake libtool; \
    \
        pip --quiet --quiet install --no-cache-dir \
            # Get dodoo-* devops toolchain
            # dodoo-scaffolder \
            # dodoo-translator \
            dodoo-tester \
            dodoo-initializer \
            dodoo-loader \
            dodoo-migrator \
            # Custom Odoo Scaffolding dependency
            pick \
            # General develpoment libraries
            astor \
            pylint-odoo \
            pylint-mccabe \
            coverage \
            ptpython \
            pudb \
            pyinotify \
            watchdog \
            wdb \
            jq \
            # Browser testing, js test recording
            websocket-client \
            ffmpeg; \
        find /usr/local -depth \
            \( \
                \( -type d -a \( -name test -o -name tests \) \) \
                -o \
                \( -type f -a \( -name '*.pyc' -o -name '*.pyo' \) \) \
            \) -exec rm -rf '{}' +; \
    \
    apt-mark auto '.*' > /dev/null; \
    [ -z "$savedAptMark" ] || apt-mark manual $savedAptMark; \
    apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
    rm -rf /var/lib/apt/lists/*;

COPY templates /templates

# Prepare odoo owned testlog folder for chrome screenshots
# Prepare odoo owned loading log folder for dodoo loader
RUN mkdir -p /var/log/dodoo/tests; \
    chown odoo:odoo /var/log/dodoo/tests; \
    mkdir -p /var/log/dodoo/loader; \
    chown odoo:odoo /var/log/dodoo/loader

# Add odoo-migrate code transformation library
RUN pip --quiet --quiet install --no-cache-dir odoo-migrate; \
    find /usr/local -depth \
        \( \
            \( -type d -a \( -name test -o -name tests \) \) \
            -o \
            \( -type f -a \( -name '*.pyc' -o -name '*.pyo' \) \) \
        \) -exec rm -rf '{}' +;

# Phantom for older odoo version
# Size: ca. 68MB
RUN export DOWNLOAD_DIR="$(mktemp -d)" && \
    cd ${DOWNLOAD_DIR} && \
    wget -P ${DOWNLOAD_DIR} -q https://bitbucket.org/ariya/phantomjs/downloads/phantomjs-2.1.1-linux-x86_64.tar.bz2 && \
    tar xf ${DOWNLOAD_DIR}/phantomjs-2.1.1-linux-x86_64.tar.bz2 && \
    cp ${DOWNLOAD_DIR}/phantomjs-2.1.1-linux-x86_64/bin/phantomjs /usr/bin/phantomjs && \
    chmod +x /usr/bin/phantomjs && \
    rm -rf ${DOWNLOAD_DIR}
