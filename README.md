# llamacpp-macOS

```
git clone https://github.com/ggerganov/llama.cpp.git
cd llama.cpp

mkdir build
cd build

cmake .. -DLLAMA_METAL=ON

cmake --build . --config Release

cd bin

./llama-cli -m ../llama-2-7b-chat.Q4_K_M.gguf -p "日本の首都はどこですか？" -ngl 9999

```
