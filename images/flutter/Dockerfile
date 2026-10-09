# ci-flutter: Flutter + Android SDK + JDK 17 (+ Node 22 untuk firebase-tools) untuk build APK/AAB.
# Semua sudah terpasang di image, karena di kontainer cache tools tidak bertahan antar job.
FROM docker.io/library/debian:bookworm

ARG FLUTTER_VERSION=3.44.6
ARG ANDROID_API=37
ARG CMDLINE_TOOLS=11076708

ENV ANDROID_HOME=/opt/android-sdk ANDROID_SDK_ROOT=/opt/android-sdk \
    FLUTTER_HOME=/opt/flutter PUB_CACHE=/opt/pub-cache \
    JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64 \
    LANG=C.UTF-8
ENV PATH=$FLUTTER_HOME/bin:$PUB_CACHE/bin:$ANDROID_HOME/cmdline-tools/latest/bin:$ANDROID_HOME/platform-tools:$PATH

RUN apt-get update -qq \
    && apt-get install -y -qq --no-install-recommends ca-certificates curl git unzip xz-utils zip openjdk-17-jdk-headless \
       libglu1-mesa file openssh-client rsync \
    && curl -fsSL https://deb.nodesource.com/setup_22.x | bash - >/dev/null \
    && apt-get install -y -qq --no-install-recommends nodejs \
    && rm -rf /var/lib/apt/lists/* \
    && git config --system --add safe.directory '*'

# Android SDK: cmdline-tools, platform, build-tools terbaru yang tersedia
RUN mkdir -p $ANDROID_HOME/cmdline-tools \
    && curl -fsSL -o /tmp/ct.zip https://dl.google.com/android/repository/commandlinetools-linux-${CMDLINE_TOOLS}_latest.zip \
    && unzip -q /tmp/ct.zip -d $ANDROID_HOME/cmdline-tools && mv $ANDROID_HOME/cmdline-tools/cmdline-tools $ANDROID_HOME/cmdline-tools/latest \
    && rm /tmp/ct.zip \
    && yes | sdkmanager --licenses >/dev/null \
    && BT=$(sdkmanager --list 2>/dev/null | grep -oE 'build-tools;[0-9]+\.[0-9]+\.[0-9]+ ' | sort -V | tail -1 | tr -d ' ') \
    && PF=$(sdkmanager --list 2>/dev/null | grep -oE "platforms;android-${ANDROID_API}(\.[0-9]+)?([^0-9 ][^ ]*)? " | sort -V | head -1 | tr -d ' ') \
    && echo "build-tools: $BT | platform: $PF" && [ -n "$PF" ] \
    && sdkmanager --install "platform-tools" "$PF" "$BT" >/dev/null \
    && sdkmanager --list_installed

# Flutter (versi dipatok) + artefak Android
RUN git clone --depth 1 -b ${FLUTTER_VERSION} https://github.com/flutter/flutter.git $FLUTTER_HOME \
    && flutter config --no-analytics --no-cli-animations \
    && flutter precache --android --no-ios --no-web --no-linux --no-windows --no-macos \
    && yes | flutter doctor --android-licenses >/dev/null \
    && flutter --version \
    && chmod -R a+rwX $FLUTTER_HOME $PUB_CACHE $ANDROID_HOME

WORKDIR /workspace
