# gpt-2

This is a C++ example running GPT-2 inference using the [ggml](https://github.com/ggerganov/ggml) library.

The program runs on the CPU - no video card is required.


The example supports the following GPT-2 models:

| Model | Description  | Disk Size |
| ---   | ---          | ---       |
| 117M  | Small model  | 240 MB    |
| 345M  | Medium model | 680 MB    |
| 774M  | Large model  | 1.5 GB    |
| 1558M | XL model     | 3.0 GB    |

Sample performance on MacBook M1 Pro:

| Model | Size  | Time / Token |
| ---   | ---   | ---    |
| GPT-2 |  117M |   5 ms |
| GPT-2 |  345M |  12 ms |
| GPT-2 |  774M |  23 ms |
| GPT-2 | 1558M |  42 ms |




