# InternVL

We made the following changes to the original repo:

* We have added support for Internvl models whose language model is the Qwen series model
* We have added support for running internvl models without inputting images, but only for dialogue

Currently this implementation supports all InternVL models, we take [InternVL-4B](https://huggingface.co/OpenGVLab/InternVL2_5-4B) as example. 

## Usage
1. Install common

```sh
cd llama.cpp/common/
mkdir build
cd build
cmake ..
cp ../../include/llama.h ../
cp ../../ggml/include/ggml.h ../
cp ../../ggml/include/ggml-backend.h ../
cp ../../ggml/include/ggml-alloc.h ../
make
```

2. Install ggml

```sh
cd llama.cpp/ggml/
mkdir build
cd build
cmake ..
make 
```

3. Install llama

```sh
cd llama.cpp/
rm -rf common/llama.h
rm -rf common/ggml.h
rm -rf common/ggml-backend.h
rm -rf common/ggml-alloc.h
mkdir build
cd build
cmake ..
make
```

4. Install Internvl

```sh
cd llama.cpp/examples/internvl/
mkdir build
cd build
cmake ..
make
/usr/bin/c++ -rdynamic "CMakeFiles/llama-internvl-cli.dir/internvl-cli.o" CMakeFiles/internvl.dir/internvl.o CMakeFiles/internvl.dir/clip.o -o llama-internvl-cli -L ../../../common/build/ -L ../../../ggml/build/src/ -L ../../../build/src/  -lcommon -lggml -lllama -lpthread
```

5. Run Internvl

Add Environment Variable

```sh
export LD_LIBRARY_PATH=../../../common/build/:$LD_LIBRARY_PATH
export LD_LIBRARY_PATH=../../../ggml/build/src/:$LD_LIBRARY_PATH
export LD_LIBRARY_PATH=../../../build/src/:$LD_LIBRARY_PATH
```

Run Internvl. For example:

With image

```sh
./llama-internvl-cli -m InternVL2_5-4B-gguf/Qwen2.5-3B-Instruct-F16.gguf --mmproj InternVL2_5-2B-gguf/model.safetensors.gguf --image path/to/an/image.jpg -p "<image>\nPlease describe the image shortly."
```

Without image

```sh
./llama-internvl-cli -m InternVL2_5-4B-gguf/Qwen2.5-3B-Instruct-F16.gguf --mmproj InternVL2_5-2B-gguf/model.safetensors.gguf -p "Hello!"
```

## Model conversation

1. Clone `InternVL-4B` locally:

```sh
git clone https://huggingface.co/OpenGVLab/InternVL2_5-4B
```

2. Convert language model

```sh
mkdir InternVL2_5-4B-chat
cd llama.cpp/examples/internvl/
python split_language_tensors.py -m InternVL2_5-4B/model-00001-of-00002.safetensors -o InternVL2_5-4B-chat/model-00001-of-00002.safetensors
python split_language_tensors.py -m InternVL2_5-4B/model-00002-of-00002.safetensors -o InternVL2_5-4B-chat/model-00002-of-00002.safetensors
cp InternVL2_5-4B/merges.txt InternVL2_5-4B-chat/
cp InternVL2_5-4B/*.json InternVL2_5-4B-chat/
```

Replace the `InternVL2_5-4B-chat/config.json` with `llm_config` item in `InternVL2_5-4B/config.json`

Delete all parameters except `language_model.XXX` in the `InternVL2_5-4B-chat/model.safetensors.index.json` file (content after line 441). Then delete the `language_model.` from the parameter name for all the remaining parameters

The examples of these two files can be found in `llama.cpp/example/internvl/json_example/`

Use `convert_hf_to_gguf.py` to convert the language model to GGUF:

```sh
cd llama.cpp
python convert_hf_to_gguf.py InternVL2_5-4B-chat/
```

3. Convert vision model

```sh
python vision_model_to_gguf_modify.py -m InternVL2_5-4B/model-00001-of-00002.safetensors,/workspace/InternVL2_5-4B/model-00002-of-00002.safetensors
```

4. Collect models

```sh
mkdir InternVL2_5-4B-gguf
cp InternVL2_5-4B/model.safetensors.gguf InternVL2_5-4B-gguf/
cp InternVL2_5-4B-chat/Qwen2.5-3B-Instruct-F16.gguf InternVL2_5-4B-gguf/
```

