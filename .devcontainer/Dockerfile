FROM mcr.microsoft.com/devcontainers/base:ubuntu-24.04
# RUN useradd -u 3333 -G sudo -m -d /home/gitpod -s /bin/bash gitpod && echo "gitpod:gitpod" | chpasswd \
#     && sed -i.bkp -e '/Defaults\tuse_pty/d' -e 's/%sudo\s\+ALL=(ALL\(:ALL\)\?)\s\+ALL/%sudo ALL=NOPASSWD:ALL/g' /etc/sudoers
USER vscode
ENV HOME=/home/vscode
# SHELL ["/bin/bash", "-c"]

# # use this Dockerfile to install additional tools you might need, e.g.
# # RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
# #     && apt-get -y install --no-install-recommends <your-package-list-here>

ENV ANDROID_HOME=$HOME/android-sdk
ENV PATH="$ANDROID_HOME/emulator:$ANDROID_HOME/tools:$ANDROID_HOME/tools/bin:$ANDROID_HOME/cmdline-tools/latest/bin:$ANDROID_HOME/cmdline-tools/bootstrap/bin:$ANDROID_HOME/platform-tools:$PATH"
ENV PATH="$HOME/.local/bin:$HOME/.bun/bin:$PATH"

# Install Android SDK
RUN sudo apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && sudo apt-get -y install --no-install-recommends \
        curl \
        unzip \
        openjdk-21-jdk

RUN version="9477386" && mkdir -p $ANDROID_HOME/cmdline-tools/bootstrap \
    && curl -L https://dl.google.com/android/repository/commandlinetools-linux-${version}_latest.zip -o /tmp/cmdline-tools.zip \
    && unzip -q /tmp/cmdline-tools.zip -d /tmp/unzip && mv /tmp/unzip/*/* $ANDROID_HOME/cmdline-tools/bootstrap

RUN yes | sdkmanager --licenses \
    && sdkmanager "cmdline-tools;latest" \
    && sdkmanager \
        "platform-tools" \
        "build-tools;35.0.0" \
        "ndk;28.1.13356709" \
        "platforms;android-35" \
        "emulator" \
        "add-ons;addon-google_apis-google-24" \
        "cmake;4.0.2" \
        "system-images;android-35;google_apis;x86_64"

# Cold boot the emulator to start it quickly later
RUN avdmanager create avd --name headless_avd --package "system-images;android-35;google_apis;x86_64" --device "pixel" && avdmanager list avd
# RUN if test -e /dev/kvm; then sudo chmod 777 /dev/kvm; (until adb shell getprop sys.boot_completed | grep -q 1; do sleep 2; done && sleep 10 && adb emu kill) & emulator -avd headless_avd -no-window -no-audio -no-boot-anim -gpu swiftshader_indirect -accel off -no-metrics; fi


USER root
# Install necessary kernel tools
RUN export DEBIAN_FRONTEND=noninteractive \
    && apt-get -y install --no-install-recommends \
        kmod \
        cpu-checker \
        qemu-system-x86 qemu-utils qemu-kvm \
        file

# Install scrcpy
RUN curl -L https://github.com/Genymobile/scrcpy/releases/download/v3.2/scrcpy-linux-x86_64-v3.2.tar.gz -o /tmp/scrcpy.tar.gz \
    && tar -xf /tmp/scrcpy.tar.gz -C /tmp \
    && mv /tmp/scrcpy-linux*/scrcpy /usr/local/bin/scrcpy \
    && chmod +x /usr/local/bin/scrcpy

    
USER vscode
# Install uv & mysc
RUN curl -LsSf https://astral.sh/uv/install.sh | sh \
    && curl -fsSL https://fnm.vercel.app/install | bash
    # && sudo apt-get install -y portaudio19-dev \
    # && uv tool install mysc[web]

# Install ws-scrcpy
SHELL ["/bin/bash", "-lic"]
RUN fnm install --lts \
    && cd $HOME && git clone https://github.com/NetrisTV/ws-scrcpy.git && cd ws-scrcpy \
    && uv run npm install && npm run dist:prod
    # && uv run bun install && uv run bun run dist:prod