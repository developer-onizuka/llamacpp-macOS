# llamacpp-macOS

If you want to use Apple Silicon GPUs in a Docker container, you'll need to use a special feature in Docker Desktop. However, Docker containers are virtualized environments that typically run on the Linux kernel. Because Linux-based containers do not have the libraries and drivers required to run Metal, the cmake command cannot find the components required for building, resulting in an error.<br>

Instead, you should build and run llama.cpp directly in your Mac's local environment. So, I decided to build the environment without any virtual machines in person.<br>


# 1. Build llama.cpp
```
git clone https://github.com/ggerganov/llama.cpp.git
cd llama.cpp

mkdir build
cd build

cmake .. -DLLAMA_METAL=ON

cmake --build . --config Release
```

# 2. Try inference by using gpt-oss-20b
```
cd bin

./bin/llama-cli -m ../../Docker/models/gpt-oss-20b-Q4_K_M.gguf -p "Please create readme.md file which explains how to use llama.cpp." -ngl 9999
```

You can see the Metal GPUs are actually used by llama-cli process.<br>
<img src="https://github.com/developer-onizuka/llamacpp-macOS/blob/main/gpt-oss-20b.png" width="800">

The following is the result of the command above.


-----

**llama.cpp** is a lightweight, header‑only implementation of the LLaMA (Large Language Model) architecture in pure C++.  
It is designed to be fast, portable and easy to embed into any C++ project – no external libraries or build tools required.

> **TL;DR**  
> ```bash
> # build (C++17 or newer)
> g++ -O3 -march=native -std=c++17 -o llama llama.cpp
> # run
> ./llama -m models/llama-7b.bin -p "Hello world" -n 20
> ```

---

## Table of Contents

1. [What is llama.cpp?](#what-is-llamacpp)  
2. [Features](#features)  
3. [Getting Started](#getting-started)  
   1. [Requirements](#requirements)  
   2. [Download](#download)  
   3. [Build](#build)  
4. [Usage](#usage)  
   1. [Command‑line options](#command-line-options)  
   2. [API usage](#api-usage)  
5. [Model Files](#model-files)  
6. [Performance](#performance)  
7. [Contributing](#contributing)  
8. [License](#license)  

---

## What is llama.cpp?

- **Pure C++** – no external dependencies, no Python or CUDA.  
- **Header‑only** – copy the single `llama.cpp` file into your project.  
- **Fast** – uses SIMD where available and a highly‑optimized kernel.  
- **Cross‑platform** – works on Windows, macOS, Linux, Android, Raspberry‑Pi…  
- **Model‑agnostic** – accepts any binary model that follows the original LLaMA format.  

---

## Features

| Feature | Status |
|---------|--------|
| **Inference** | ✅ |
| **Sampling** | ✅ (temperature, top‑k, top‑p, repetition penalty) |
| **Tokenization** | ✅ (BPE + vocab) |
| **Memory‑efficient** | ✅ (int8, int4 quantization) |
| **Multithreading** | ✅ (OpenMP or `std::thread`) |
| **GPU support** | ❌ (only CPU; see `llama-gpu` for CUDA) |
| **CLI demo** | ✅ (see `llama.cpp` main) |
| **C API** | ❌ (only C++) |

---

## Getting Started

### Requirements

| Item | Minimum | Notes |
|------|---------|-------|
| Compiler | GCC 8 / Clang 10 / MSVC 2019 | C++17 standard |
| CPU | x86_64 (AVX‑512 recommended) | Works on ARM as well |
| RAM | 8 GB+ | Depends on model size |
| Disk | 4 GB free | Model files can be large |

### Download

```bash
# Clone the repository
git clone https://github.com/yourname/llama.cpp
cd llama.cpp
```

You can also download the single file directly:

```bash
curl -L https://raw.githubusercontent.com/yourname/llama.cpp/main/llama.cpp -o llama.cpp
```

### Build

The easiest way is to use the included `Makefile` (Linux/macOS) or `CMakeLists.txt` (Windows).

```bash
# Linux / macOS
make

# Windows (MSVC)
cmake . -A x64
cmake --build . --config Release
```

If you prefer manual compilation:

```bash
# g++
g++ -O3 -march=native -std=c++17 -o llama llama.cpp

# clang++
clang++ -O3 -march=native -std=c++17 -o llama llama.cpp
```

The executable is named **`llama`** (or `llama.exe` on Windows).

---

## Usage

### Command‑line options

| Option | Argument | Description |
|--------|----------|-------------|
| `-m` | `<model_path>` | Path to a binary LLaMA model |
| `-p` | `<prompt>` | Prompt string to start generation |
| `-n` | `<count>` | Number of tokens to generate (default 20) |
| `-t` | `<temperature>` | Temperature for sampling (default 1.0) |
| `-k` | `<top_k>` | Top‑k sampling (default 40) |
| `-s` | `<seed>` | Random seed (default 0) |
| `-l` | `<length>` | Maximum output length (tokens) |
| `-q` | `<quantization>` | 8‑bit, 4‑bit, or raw (default `int8`) |
| `-h` | | Show help |

#### Example

```bash
./llama -m models/llama-7b.bin -p "Explain quantum computing in simple terms:" -n 100 -t 0.8 -k 50
```

The program prints the prompt followed by the generated text, one token at a time.

### API usage

```cpp
#include "llama.cpp"

int main() {
    // Load model
    auto model = llama::Model::load("models/llama-7b.bin");

    // Create session
    llama::Session sess(&model);
    sess.set_temperature(0.7f);
    sess.set_top_k(40);

    // Tokenize prompt
    auto tokens = sess.tokenize("Hello world");

    // Generate
    for (size_t i = 0; i < 30; ++i) {
        auto next = sess.next_token(tokens);
        std::cout << model.id_to_token(next);
    }
}
```

All public types are in the `llama` namespace.

---

## Model Files

Download the official LLaMA weights from the Meta AI repository or use any compatible binary.  
Place the file in a folder named `models/` (or any folder you prefer) and refer to it with the `-m` option.

> **Note**: The model file is typically several GB. Ensure you have enough disk space.

---

## Performance

On a modern Intel i7‑11700K (AVX‑512) the following speeds were measured:

| Model | Tokens/s | Memory | Notes |
|-------|----------|--------|-------|
| 7B |  110 | 5.3 GB | CPU only |
| 13B |  63 | 9.7 GB | CPU only |
| 30B |  28 | 17.1 GB | CPU only |

*All numbers are averages over 10 runs.  
Quantization reduces memory usage by 8‑bit → 4‑bit.*

---

## Contributing

Feel free to open issues or submit pull‑requests.  
If you find a bug or have a feature request, let us know!

---

## License

`llama.cpp` is released under the **MIT License**.  
See the included `LICENSE` file for details.

---

### Happy generating! 🚀

---
