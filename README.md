# Nano ggml

**This is Nano ggml, a lightweight framework designed for mobile, watches, and IoT devices. It’s based on GGML, with minimal code and <u>_no third-party libraries._</u>**

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
git clone git@github.com:manyuanbin/ggml-on-device.git
cd ggml-on-device

# install python dependencies in a virtual environment
python3.10 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

# build the examples
mkdir build && cd build
cmake ..
cmake --build . --config Release -j 8
```

## GPT-2 inference
>**Note**: The throughput on my laptop is quite good, achieving <u>**_8.04_**</u> ms per token. 

```bash
# run the GPT-2 small 117M model with q8
(base) yuanbin.myb@macbookpro build % ./bin/gpt-2-backend -m ../../ggml-on-device/build/models/gpt-2-117M/ggml-model-q8_0.bin -p "How is Boston"
main: seed = 1732260396
gpt2_model_load: loading model from '../../ggml-on-device/build/models/gpt-2-117M/ggml-model-q8_0.bin'
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
main: prompt: 'How is Boston'
main: number of tokens in prompt = 3, first 8 tokens: 2437 318 6182 

How is Boston?"

"Boston, you can do a lot more."

The question came in the midst of a series of tweets from the Boston Globe that were both funny and in need of a response. One was from Matt Zoller Seitz:

"What's up with Boston? You're here, aren't you? I know I should give you a break, because I love you. You're doing what you do."

Another, a direct quote from a New York Times op-ed by Boston Mayor Marty Walsh, was from another, more subdued source:

"I don't understand. You are right. We are here because of what you are doing. You are the best that you can be. You are a true patriot. You are a true human being. This is not about who you are. This is about who we are. This is not just a place. This is about all of us. This is a city that has been built on the

main:     load time =   210.87 ms
main:   sample time =    32.18 ms
main:  predict time =  1623.66 ms / 8.04 ms per token
main:    total time =  1871.03 ms

```

## Model Quantization

**How do you quantize a model? Reach out to  [Yuanbin](mailto:ybinman@bu.edu)!**

```
240M	models/gpt-2-117M/ggml-model-f16.bin
 70M	models/gpt-2-117M/ggml-model-q4_0.bin
 78M	models/gpt-2-117M/ggml-model-q4_1.bin
 70M	models/gpt-2-117M/ggml-model-q4_k.bin
129M	models/gpt-2-117M/ggml-model-q8_0.bin
240M	models/gpt-2-117M/ggml-model.bin
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

