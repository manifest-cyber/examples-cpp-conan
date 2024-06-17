FROM gcc:13.2.0

RUN apt update && apt install -y \
    python3-pip \
    pipx \
    cmake \
    clang \
    clang-tools \
    clangd

RUN pipx ensurepath && \
    pipx install "conan==1.64.0"
