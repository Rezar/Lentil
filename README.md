# nanoLLM

**This is nanoLLM, a lightweight framework designed for mobile, watches, and IoT devices. It’s based on GGML, with minimal code and <u>_no third-party libraries._</u>**

- [Introduction to ggml](https://huggingface.co/blog/introduction-to-ggml)
- [The GGUF file format](https://github.com/ggerganov/ggml/blob/master/docs/gguf.md)


## Features

- **_No third-party dependencies_**
- Integer quantization support
- Broad hardware support
- Automatic differentiation
- ADAM and L-BFGS optimizers
- Zero memory allocations during runtime

## Build

```bash
git clone git@github.com:Rezar/nanoLLM.git
cd nanoLLM

# install python dependencies in a virtual environment
cd ./apps/gpt-2
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

# build the examples
mkdir build && cd build
cmake ..
cmake --build . --config Release -j 8
```

## Model Quantization

- Download model 
```
./download-model.sh 117M
```

- Convert checkpoint to ggml model
```
python convert-ckpt-to-ggml.py ../../models/gpt-2-117M/ 1
```

- Quantize model (fp16 -> q8, quantize to q4 can get bad performance)
```
cd ../../build/
./bin/gpt-2-quantize ../models/gpt-2-117M/ggml-model-f16.bin ../models/gpt-2-117M/ggml-model-q8.bin 7
```

- Check the models
```
240M	models/gpt-2-117M/ggml-model-f16.bin
 70M	models/gpt-2-117M/ggml-model-q4_0.bin
 78M	models/gpt-2-117M/ggml-model-q4_1.bin
 70M	models/gpt-2-117M/ggml-model-q4_k.bin
129M	models/gpt-2-117M/ggml-model-q8_0.bin
240M	models/gpt-2-117M/ggml-model.bin
```


## GPT-2 inference
>**Note**: The throughput on my laptop is quite good, achieving <u>**_3.76_**</u> ms per token. 

```bash
# run the GPT-2 small 117M model with q8
(base) yuanbin.myb@macbookpro build % ./bin/gpt-2-backend -m ../../ggml-on-device/build/models/gpt-2-117M/ggml-model-q8_0.bin -p "How is Boston"
main: seed = 1732311145
gpt2_model_load: loading model from '../models/gpt-2-117M/ggml-model-q8.bin'
gpt2_model_load: n_vocab = 50257
gpt2_model_load: n_ctx   = 1024
gpt2_model_load: n_embd  = 768
gpt2_model_load: n_head  = 12
gpt2_model_load: n_layer = 12
gpt2_model_load: ftype   = 2007
gpt2_model_load: qntvr   = 2
gpt2_model_load: using CPU backend
gpt2_model_load: ggml tensor size    = 336 bytes
gpt2_model_load: backend buffer size = 167.75 MB
gpt2_model_load: memory size =   144.00 MB, n_mem = 24576
gpt2_model_load: model size  =   128.64 MB
extract_tests_from_file : No test file found.
test_gpt_tokenizer : 0 tests failed out of 0 tests.
main: compute buffer size: 9.47 MB
main: prompt: 'How is Boston?'
main: number of tokens in prompt = 4, first 8 tokens: 2437 318 6182 30 

How is Boston?

There are many questions about Boston, but I'd like to start with one of the most important ones.

Why didn't Boston do a better job of preparing for the World Cup than at home?

The American teams lost three matches, including an impressive 2-1 win in the final. Boston did well in both, but the home series was much tighter.

There were two goals in the World Cup, two in Brazil, and two in China. What was your impression of these teams, what was your favorite?

I like the Brazilian teams. I like the U.S. teams, but I didn't like the U.S. team. I thought the U.S. teams were too good. They had an outstanding defense, and they had a solid goal scorer. They had a great team, but they were slow in the beginning. I think that was something that was kind of a surprise. I think it was more of an expectation for

main:     load time =   140.10 ms
main:   sample time =    21.43 ms
main:  predict time =   762.94 ms / 3.76 ms per token
main:    total time =   926.81 ms

```



## Compiling for Android

Download and unzip the NDK from this download [page](https://developer.android.com/ndk/downloads). Set the NDK_ROOT_PATH environment variable or provide the absolute path to the CMAKE_ANDROID_NDK in the command below.

```bash
cmake .. \
   -DCMAKE_SYSTEM_NAME=Android \
   -DCMAKE_SYSTEM_VERSION=33 \
   -DCMAKE_ANDROID_ARCH_ABI=arm64-v8a \
   -DCMAKE_ANDROID_NDK=$NDK_ROOT_PATH
   -DCMAKE_ANDROID_STL_TYPE=c++_shared
```

```bash
# create directories
adb shell 'mkdir /data/local/tmp/bin'
adb shell 'mkdir /data/local/tmp/models'

# push the compiled binaries to the folder
adb push bin/* /data/local/tmp/bin/

# push the ggml library
adb push src/libggml.so /data/local/tmp/

# push model files
adb push models/gpt-2-117M/ggml-model.bin /data/local/tmp/models/

adb shell
cd /data/local/tmp
export LD_LIBRARY_PATH=/data/local/tmp
./bin/gpt-2-backend -m models/ggml-model.bin -p "this is an example"
```

