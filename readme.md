# 임베딩 모델 양자화 성능 비교

## 실험 환경
```
$ nvidia-smi                 
Wed Apr 29 00:01:49 2026       
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 580.126.20             Driver Version: 580.126.20     CUDA Version: 13.0     |
+-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA GeForce RTX 3090        On  |   00000000:03:00.0 Off |                  N/A |
|  0%   37C    P8             27W /  240W |   10391MiB /  24576MiB |      0%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+
```

```
$ docker image list
IMAGE                                                ID             DISK USAGE   CONTENT SIZE   EXTRA
ghcr.io/ggml-org/llama.cpp:server-cuda               1f0b553ce3a3       3.61GB             0B
```

```
docker run --gpus all -p 8111:8080 \
  --rm \
  -v ./:/models \
  -v ./cache:/root/.cache \
  ghcr.io/ggml-org/llama.cpp:server-cuda \
  --model /models/mradermacher/Qwen3-Embedding-0.6B-i1-GGUF/Qwen3-Embedding-0.6B.i1-Q4_K_M.gguf \
  --embedding \
  --metrics
```


## 대상 모델

[Qwen/Qwen3-Embedding-0.6B-GGUF](https://huggingface.co/Qwen/Qwen3-Embedding-0.6B-GGUF)
- Qwen3-Embedding-0.6B-f16.gguf (baseline)
- Qwen3-Embedding-0.6B-Q8_0.gguf

[mradermacher/Qwen3-Embedding-0.6B-i1-GGUF](https://huggingface.co/mradermacher/Qwen3-Embedding-0.6B-i1-GGUF)
- Qwen3-Embedding-0.6B.i1-Q6_K.gguf
- Qwen3-Embedding-0.6B.i1-Q5_K_M.gguf
- Qwen3-Embedding-0.6B.i1-Q4_K_M.gguf
- Qwen3-Embedding-0.6B.i1-Q3_K_M.gguf
- Qwen3-Embedding-0.6B.i1-Q2_K.gguf


## 실험 방법

1. 단어, 문장, 문단 (document) 3가지 유형의 한국어 샘플 구성
  - 단어 샘플 - binjang/NIKL-korean-english-dictionary 에서 추출
  - 문장 샘플 - klue/klue 에서 추출
  - 문단 샘플 - heegyu/namuwiki-extracted 에서 추출

3. 각 모델에 대해서 임베딩 생성
4. Qwen3-Embedding-0.6B-f16.gguf 모델과 비교하여 양자화 된 모델의 정확도 비교

샘플 생성 및 시각화 코드는 AI으로 작성됨

## 결론
- 짧은 입력보다 긴 텍스트 입력일 때의 오차가 적다. 입력 텍스트가 많을수록 문맥을 파악하기 용이하기 때문으로 추측
- Q8 양자화 모델은 f16과 99% 유사하지만 top-k 비교에서는 97% 를 넘지 못했다.
- 문장 및 문서 검색용으로 top-5 오차 3% 이내 허용한다면 Q4 양자화 모델을 사용해도 된다.
- 문서 검색용으로 top-5 오차 2% 이내 허용한다면 Q4 양자화 모델을 사용해도 된다.
- 하드웨어 자원이 모자란 상황에 1% ~ 5%의 오차를 허용한다면 Q8과 f16 모델을 섞어서 사용해도 괜찮을 수도 있다

## 한계
- 한국어 샘플만 사용하여 다국어일때의 결과는 확인하지 못함
- f16, Q8 모델과 Q6, Q5, Q4, Q3, Q2 모델 양자화 주체가 달라서 세부 수치가 다를 수 있음
- 더 큰 사이즈의 모델 (eg. Qwen/Qwen3-Embedding-4B, Qwen/Qwen3-Embedding-8B) 을 사용한다면 양자화 모델 사이의 유사도는 이 결과와 다를 수 있음



## Data License

This repository contains code and optional data samples.

The code is licensed under the MIT License.

The files under `sample/` are derived from third-party datasets and are
licensed under their respective original licenses:

- `sample/words.json`
  - Source: `binjang/NIKL-korean-english-dictionary`
  - License shown on Hugging Face: MIT
  - Original data source: National Institute of Korean Language Korean Basic Dictionary

- `sample/sentences.json`
  - Source: `klue/klue`, STS split
  - License: CC-BY-SA-4.0

- `sample/documents.json`
  - Source: `heegyu/namuwiki-extracted`
  - License: CC-BY-NC-SA-2.0
  - Non-commercial use only
