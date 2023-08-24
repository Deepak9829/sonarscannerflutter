FROM alpine:3.17



ENV UID=1000
ENV GID=1000
ENV USER="developer"
ENV JAVA_VERSION="8"
ENV ANDROID_TOOLS_URL="https://dl.google.com/android/repository/commandlinetools-linux-6858069_latest.zip"
ENV ANDROID_VERSION="29"
ENV ANDROID_BUILD_TOOLS_VERSION="29.0.3"
ENV ANDROID_ARCHITECTURE="x86_64"
ENV ANDROID_SDK_ROOT="/home/$USER/android"
ENV FLUTTER_CHANNEL="stable"
ENV FLUTTER_VERSION="2.2.1"
ENV FLUTTER_URL="https://storage.googleapis.com/flutter_infra/releases/$FLUTTER_CHANNEL/linux/flutter_linux_$FLUTTER_VERSION-$FLUTTER_CHANNEL.tar.xz"
ENV FLUTTER_HOME="/home/$USER/flutter"
ENV FLUTTER_WEB_PORT="8090"
ENV FLUTTER_DEBUG_PORT="42000"
ENV FLUTTER_EMULATOR_NAME="flutter_emulator"
ENV PATH="$ANDROID_SDK_ROOT/cmdline-tools/tools/bin:$ANDROID_SDK_ROOT/emulator:$ANDROID_SDK_ROOT/platform-tools:$ANDROID_SDK_ROOT/platforms:$FLUTTER_HOME/bin:$PATH"

RUN apk update && \
    apk add openjdk${JAVA_VERSION} curl unzip sed git bash xz xauth xvfb sudo && \
    rm -rf /var/cache/apk/* && \
    apk add --no-cache unzip

RUN addgroup -g $GID $USER && \
    adduser -s /bin/bash -u $UID -G $USER -D $USER && \
    echo "$USER ALL=(root) NOPASSWD:ALL" > /etc/sudoers.d/$USER && \
    chmod 0440 /etc/sudoers.d/$USER

USER $USER
WORKDIR /home/$USER



RUN mkdir -p $ANDROID_SDK_ROOT && \
    mkdir -p /home/$USER/.android && \
    touch /home/$USER/.android/repositories.cfg && \
    curl -o android_tools.zip $ANDROID_TOOLS_URL && \
    unzip -qq -d "$ANDROID_SDK_ROOT" android_tools.zip && \
    rm android_tools.zip && \
    mkdir -p $ANDROID_SDK_ROOT/cmdline-tools/tools && \
    mv $ANDROID_SDK_ROOT/cmdline-tools/bin $ANDROID_SDK_ROOT/cmdline-tools/tools && \
    mv $ANDROID_SDK_ROOT/cmdline-tools/lib $ANDROID_SDK_ROOT/cmdline-tools/tools && \
    yes "y" | $ANDROID_SDK_ROOT/cmdline-tools/tools/bin/sdkmanager "build-tools;$ANDROID_BUILD_TOOLS_VERSION" && \
    yes "y" | $ANDROID_SDK_ROOT/cmdline-tools/tools/bin/sdkmanager "platforms;android-$ANDROID_VERSION" && \
    yes "y" | $ANDROID_SDK_ROOT/cmdline-tools/tools/bin/sdkmanager "platform-tools" && \
    yes "y" | $ANDROID_SDK_ROOT/cmdline-tools/tools/bin/sdkmanager "emulator" && \
    yes "y" | $ANDROID_SDK_ROOT/cmdline-tools/tools/bin/sdkmanager "system-images;android-$ANDROID_VERSION;google_apis_playstore;$ANDROID_ARCHITECTURE"

RUN curl -o flutter.tar.xz $FLUTTER_URL && \
    mkdir -p $FLUTTER_HOME && \
    tar xf flutter.tar.xz -C /home/$USER && \
    rm flutter.tar.xz && \
    flutter config --no-analytics && \
    flutter precache && \
    yes "y" | flutter doctor --android-licenses && \
    flutter doctor && \
    flutter emulators --create && \
    flutter update-packages

COPY entrypoint.sh /usr/local/bin/
COPY chown.sh /usr/local/bin/
COPY flutter-android-emulator.sh /usr/local/bin/flutter-android-emulator






LABEL org.opencontainers.image.url=https://github.com/SonarSource/sonar-scanner-cli-docker

ARG SONAR_SCANNER_HOME=/opt/sonar-scanner
ARG SONAR_SCANNER_VERSION
ARG UID=1000
ARG GID=1000
ENV JAVA_HOME=/usr/lib/jvm/java-17-openjdk \
    HOME=/tmp \
    XDG_CONFIG_HOME=/tmp \
    SONAR_SCANNER_HOME=${SONAR_SCANNER_HOME} \
    SONAR_USER_HOME=${SONAR_SCANNER_HOME}/.sonar \
    PATH=${SONAR_SCANNER_HOME}/bin:${PATH} \
    NODE_PATH=/usr/lib/node_modules \
    SRC_PATH=/usr/src \
    LANG=en_US.UTF-8 \
    LC_ALL=en_US.UTF-8

WORKDIR /opt

RUN set -eux; \
    addgroup -S -g ${GID} scanner-cli; \
    adduser -S -D -u ${UID} -G scanner-cli scanner-cli; \
    apk add --no-cache --virtual build-dependencies wget unzip gnupg; \
    apk add --no-cache git python3 py-pip bash shellcheck 'nodejs>12' openjdk17-jre curl musl-locales musl-locales-lang; \
    wget -U "scannercli" -q -O /opt/sonar-scanner-cli.zip https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-${SONAR_SCANNER_VERSION}.zip; \
    wget -U "scannercli" -q -O /opt/sonar-scanner-cli.zip.asc https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-${SONAR_SCANNER_VERSION}.zip.asc; \
    for server in $(shuf -e hkps://keys.openpgp.org \
                            hkps://keyserver.ubuntu.com) ; do \
        gpg --batch --keyserver "${server}" --recv-keys 679F1EE92B19609DE816FDE81DB198F93525EC1A && break || : ; \
    done; \
    gpg --verify /opt/sonar-scanner-cli.zip.asc /opt/sonar-scanner-cli.zip; \
    unzip sonar-scanner-cli.zip; \
    rm sonar-scanner-cli.zip sonar-scanner-cli.zip.asc; \
    mv sonar-scanner-${SONAR_SCANNER_VERSION} ${SONAR_SCANNER_HOME}; \
    pip install --no-cache-dir --upgrade pip; \
    pip install --no-cache-dir pylint; \
    apk del --purge build-dependencies; \
    mkdir -p "${SRC_PATH}" "${SONAR_USER_HOME}" "${SONAR_USER_HOME}/cache"; \
    chown -R scanner-cli:scanner-cli "${SONAR_SCANNER_HOME}" "${SRC_PATH}"; \
    chmod -R 777 "${SRC_PATH}" "${SONAR_USER_HOME}";

COPY --chown=scanner-cli:scanner-cli bin /usr/bin/

VOLUME [ "/tmp/cacerts" ]

WORKDIR ${SRC_PATH}

ENTRYPOINT ["/usr/bin/entrypoint.sh","/usr/local/bin/entrypoint.sh"]

CMD ["sonar-scanner"]