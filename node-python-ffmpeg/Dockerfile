# Build stage
FROM nvidia/cuda:12.2.0-devel-ubuntu22.04 AS build

# Install build dependencies
RUN apt-get update && apt-get install -y \
    git build-essential yasm cmake libtool libc6 libc6-dev unzip wget \
    libnuma1 libnuma-dev zlib1g-dev libssl-dev pkg-config \
    libass-dev libfreetype6-dev libsdl2-dev libtool libva-dev \
    libvdpau-dev libvorbis-dev libxcb1-dev libxcb-shm0-dev \
    libxcb-xfixes0-dev texinfo zlib1g-dev libva-x11-2 \
    libfontconfig1-dev libfribidi-dev libharfbuzz-dev libmp3lame-dev \
    libopencore-amrnb-dev libopencore-amrwb-dev libopus-dev libsoxr-dev \
    libspeex-dev libtheora-dev libvidstab-dev libvo-amrwbenc-dev \
    libvpx-dev libwebp-dev libx264-dev libx265-dev libxml2-dev \
    libfdk-aac-dev libaom-dev libxvidcore-dev

# Install nv-codec-headers
RUN git clone --depth 1 --branch n12.0.16.0 https://git.videolan.org/git/ffmpeg/nv-codec-headers.git && \
    cd nv-codec-headers && \
    make install && \
    cd .. && \
    rm -rf nv-codec-headers

# Build SVT-AV1
RUN git clone --depth 1 --branch v1.4.1 https://gitlab.com/AOMediaCodec/SVT-AV1.git && \
    cd SVT-AV1 && \
    cd Build && \
    cmake .. -G"Unix Makefiles" -DCMAKE_BUILD_TYPE=Release && \
    make -j$(nproc) && \
    make install && \
    ldconfig

# Download and compile FFmpeg
RUN git clone --depth 1 --branch n6.1.2 https://github.com/FFmpeg/FFmpeg.git ffmpeg && \
    cd ffmpeg && \
    ./configure \
    --extra-cflags="-I/usr/local/cuda/include" \
    --extra-ldflags="-L/usr/local/cuda/lib64" \
    --enable-gpl \
    --enable-nonfree \
    --enable-cuda-nvcc \
    --enable-libnpp \
    --enable-shared \
    --disable-static \
    --enable-libass \
    --enable-libfdk-aac \
    --enable-libfreetype \
    --enable-libmp3lame \
    --enable-libopus \
    --enable-libvorbis \
    --enable-libvpx \
    --enable-libx264 \
    --enable-libx265 \
    --enable-libwebp \
    --enable-libxvid \
    --enable-libsoxr \
    --enable-openssl \
    --enable-libfontconfig \
    --enable-libfribidi \
    --enable-libharfbuzz \
    --enable-libsvtav1 \
    --enable-libvidstab \
    --enable-libxml2 \
    --enable-libaom \
    && \
    make -j$(nproc) && \
    make install && \
    ldconfig

# Final stage
FROM nvidia/cuda:12.2.0-base-ubuntu22.04

# Copy FFmpeg binaries and libraries from build stage
COPY --from=build /usr/local/bin/ffmpeg /usr/local/bin/ffmpeg
COPY --from=build /usr/local/bin/ffprobe /usr/local/bin/ffprobe
COPY --from=build /usr/local/lib/libav* /usr/local/lib/
COPY --from=build /usr/local/lib/libsw* /usr/local/lib/
COPY --from=build /usr/local/lib/libpostproc.so* /usr/local/lib/
COPY --from=build /usr/local/lib/libSvtAv1* /usr/local/lib/

# Install runtime dependencies, Node.js, Python 3, and pip
RUN apt-get update && \
    apt-get install -y curl software-properties-common \
    libxcb1 libxcb-shm0 libxcb-xfixes0 libxcb-shape0 \
    libasound2 libass9 libvdpau1 libva-drm2 libva2 libva-x11-2 \
    libomxil-bellagio0 libsdl2-2.0-0 libsndio7.0 libxv1 \
    libtheora0 libvorbis0a libvorbisenc2 libvorbisfile3 libzmq5 libzvbi0 \
    libmp3lame0 libopus0 libvpx7 libx264-163 libx265-199 libwebpmux3 \
    libxvidcore4 libsoxr0 libfontconfig1 libfribidi0 libharfbuzz0b \
    libvidstab1.1 libvo-amrwbenc0 libopencore-amrnb0 libopencore-amrwb0 \
    libfdk-aac2 libaom3 && \
    curl -fsSL https://deb.nodesource.com/setup_current.x | bash - && \
    add-apt-repository ppa:deadsnakes/ppa && \
    apt-get update && \
    apt-get install -y --no-install-recommends nodejs python3.10 python3-pip && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* && \
    ln -s /usr/bin/python3 /usr/bin/python && \
    ldconfig

# Verify installations
RUN node --version && python --version && pip3 --version && ffmpeg -version

WORKDIR /app
CMD ["bash"]
