parentTopic:: [[Linux]]

# Build clang format
- Clone

    ```bash
    git clone https://github.com/llvm/llvm-project.git
    cd llvm-project
    ```
- Build
    ```bash
    mkdir build
    cd build
    cmake ../llvm -DLLVM_ENABLE_PROJECTS=clang
    make clang-format
    ```
- Extract
    ```bash
    mkdir ~/clang-format-binary
    cp bin/clang-format ~/clang-format-binary/
    ```