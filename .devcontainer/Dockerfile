FROM ubuntu:22.04 AS frontend-build-base-img

ENV DEBIAN_FRONTEND="noninteractive"
ENV JAVA_VERSION="11"

# Install the necessary dependencies.
RUN apt-get update \
  && apt-get install --yes --no-install-recommends \
    openjdk-$JAVA_VERSION-jdk \
    curl \
    unzip \
    sed \
    git \
    bash \
    xz-utils \
    libglvnd0 \
    ssh \
    xauth \
    x11-xserver-utils \
    libpulse0 \
    libxcomposite1 \
    libgl1-mesa-glx \
    wget \
  && rm -rf /var/lib/{apt,dpkg,cache,log}

# Checkout Flutter
ENV FLUTTER_ROOT="/home/flutter"
ENV PATH="$PATH:$FLUTTER_ROOT/bin"
RUN mkdir -p $FLUTTER_ROOT && git clone https://github.com/flutter/flutter.git $FLUTTER_ROOT

ENV FLUTTER_VERSION="3.16.0"
RUN cd $FLUTTER_ROOT && git fetch && git checkout $FLUTTER_VERSION

#######################################################################################
FROM frontend-build-base-img AS frontend-android-build-img


# Install Gradle.
ENV GRADLE_VERSION="8.0"
ENV GRADLE_USER_HOME="/opt/gradle"
ENV GRADLE_URL="https://services.gradle.org/distributions/gradle-${GRADLE_VERSION}-bin.zip"
ENV PATH="$GRADLE_USER_HOME/bin:$PATH"
RUN curl -L $GRADLE_URL -o gradle-$GRADLE_VERSION-bin.zip \
  && apt-get install -y unzip \
  && unzip gradle-$GRADLE_VERSION-bin.zip \
  && mv gradle-$GRADLE_VERSION $GRADLE_USER_HOME \
  && rm gradle-$GRADLE_VERSION-bin.zip

#Android Studio
ENV ANDROID_STUDIO_VERSION=4.1.3
ENV ANDROID_STUDIO_URL=https://redirector.gvt1.com/edgedl/android/studio/ide-zips/4.1.3.0/android-studio-ide-201.7199119-linux.tar.gz
ENV ANDROID_STUDIO_ROOT="/usr/local/android-studio-root"
ENV ANDROID_EMULATOR_USE_SYSTEM_LIBS=1
RUN wget "$ANDROID_STUDIO_URL" -O android-studio.tar.gz
RUN mkdir -p $ANDROID_STUDIO_ROOT && tar xzvf android-studio.tar.gz -C $ANDROID_STUDIO_ROOT
RUN rm android-studio.tar.gz
RUN ln -s /studio-data/profile/AndroidStudio$ANDROID_STUDIO_VERSION .AndroidStudio$ANDROID_STUDIO_VERSION
RUN ln -s /studio-data/Android Android
RUN ln -s /studio-data/profile/android .android
RUN ln -s /studio-data/profile/java .java
RUN ln -s /studio-data/profile/gradle .gradle

# Install the Android SDK.
ENV ANDROID_TOOLS_URL="https://dl.google.com/android/repository/commandlinetools-linux-9477386_latest.zip"
ENV ANDROID_SDK_ROOT="/usr/local/android-sdk"
ENV PATH="$ANDROID_SDK_ROOT/cmdline-tools/latest/bin:$ANDROID_SDK_ROOT/emulator:$ANDROID_SDK_ROOT/platform-tools:$ANDROID_SDK_ROOT/platforms:$PATH"
RUN mkdir /root/.android \
  && touch /root/.android/repositories.cfg \
  && mkdir -p $ANDROID_SDK_ROOT \
  && curl -o android_tools.zip $ANDROID_TOOLS_URL \
  && unzip -qq -d "$ANDROID_SDK_ROOT" android_tools.zip \
  && rm android_tools.zip \
  && mv $ANDROID_SDK_ROOT/cmdline-tools $ANDROID_SDK_ROOT/latest \
  && mkdir -p $ANDROID_SDK_ROOT/cmdline-tools \
  && mv $ANDROID_SDK_ROOT/latest $ANDROID_SDK_ROOT/cmdline-tools/latest 

# Install sdkmanager tools.
RUN yes "y" | sdkmanager "platforms;android-33" 
RUN yes "y" | sdkmanager "platform-tools"
RUN yes "y" | sdkmanager "build-tools;33.0.0" 

# Config Flutter
RUN flutter config --android-sdk $ANDROID_SDK_ROOT
RUN yes "y" | flutter doctor --android-licenses


#######################################################################################
FROM frontend-android-build-img AS dev-env-img
RUN apt-get --yes --no-install-recommends install docker-compose  make 
RUN flutter config --enable-web