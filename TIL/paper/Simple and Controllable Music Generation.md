[Simple and Controllable Music Generation](https://arxiv.org/abs/2306.05284)

### MusicGen이란
1. 소개
 - 여러 스트림의 압축된 이산적 음악 표현(토큰)을 처리하는 단일 언어 모델.
 - 기존 연구와 달리 단일 단계 트랜스포머 언어로 구성됨. 
 - 이 방식은 계층적 방식이나 업샘플링 등 여러 모델을 연결해야하는 복잡성을 제거한다.

2. 특징
 - 효율적인 토큰 인터리빙(interleaving) 패턴을 도입하여 모델의 효율성을 높임.
 - 이 접근 방식은 단일 및 스테레오 샘플을 모두 생성할 수 있으며, 텍스트 설명이나 멜로디 특징을 조건으로 활용할 수 있어 생성된 음악에 대한 제어력을 향상시킨다.

> 음악 샘플, 코드, 모델은 [깃허브](https://github.com/facebookresearch/audiocraft)에 공개되어있어 연구자와 개발자가 이를 활용할 수 있다.

### Models and Hyperparameters
1. Audio Tokenization Model
- 모델 구조: EnCodec 비인과적 5층 모델
- 양자화: 잔여 벡터 양자화(Residual Vector Quantization, RVQ) 사용.
- 훈련데이터: 오디오 시퀀스에서 임의로 자른 1초 길이의 오디오 세그먼트를 사용하여 훈련

2. Transformer Model
- 모델 크기: 300M, 1.5B, 3.3B 파라미터.
- 효율성: 긴 시퀀스를 처리하기 위해 Flash Attention([Dao et al., 2022])을 사용하여 속도와 메모리 사용량 최적화.
- 훈련데이터: 30초 길이의 오디오 트랙을 임의로 샘플링하여 사용.
- 샘플링방법: Top-k 샘플링([Fan et al., 2018]): k=250, temperature = 1.0

3. Text Preprocessing
- 텍스트 정규화
- 텍스트 설명 강화
- 텍스트 증강

4. Codebook Patterns and Conditioning
- 코드북 패턴
  - "지연 패턴(Delay Interleaving Pattern)" 사용
  - 30초 길이 오디오 → 1500개의 오토레그레시브 단계로 변환.
- 텍스트 조건화
  - T5 인코더
 - 멜로디 조건화
   - 크로마그램계산
- 샘플링 시 클래스 없는 가이던스: 훈련 시 조건을 20% 확률로 드롭, 추론 시 가이던스 스케일 3.0 

### Train Datasets
1. 총 데이터량
 - 약 20,000 시간의 라이선스 음악 데이터를 사용하여 MusicGen을 훈련

2. 데이터 구성
 - 내부 데이터셋: 고품질 음악 트랩 10,000곡
 - Shutterstock 음악 데이터: 악기 연주만 포함된 음악 트랙 25,000곡
 - Pond5 음악데이터: 악기 연주만 포함된 음악 트램 365,000곡

3. 데이터 속성
 - 모든 데이터는 32kHz로 샘플링된 전체 길이의 음악으로 구성
 - 메타 데이터 포함:
     - 텍스트 설명
     - 장르, BPM, 태그 등의 정보
 - 오디오는 별로도 명시되지 않는 한 모노(mono)로 다운믹스

### Evaluation
1. 비교 대상 모델
- Riffusion, Mousai, MusicLM, Noise2Music
2. 평가 메트릭
- 객관적 평가
  - Fréchet Audio Distance (FAD): 오디오 생성의 신뢰도를 측정. (현실감)
  - Kullback-Leibler Divergence (KL): 생성된 음악과 원본 음악 간의 레이블 확률 분포 차이를 측정. (참조 음악과의 유사성)
  - CLAP Score: 생성된 오디오와 텍스트 설명 간의 정렬(alignment)을 측정.(텍스트와 오디오의 정렬도)
- 주관적 평가
  -  Overall Quality (OVL): 생성된 오디오의 전체적인 지각 품질 평가.
  - Relevance to Text Input (REL): 생성된 오디오와 텍스트 입력 간의 관련성 평가.
  
![](https://velog.velcdn.com/images/s0o0_jiiin/post/28d5d82e-34c2-4795-8aed-ed71e26e030c/image.png)


### Result
- MUSICGEN은 오디오 품질과 텍스트와의 관련성에서 기준 모델들보다 우수.
- Noise2Music은 FAD에서 가장 낮은 점수를 기록했지만, MUSICGEN은 인간 평가에서 더 나은 성과를 보임.

![](https://velog.velcdn.com/images/s0o0_jiiin/post/b0c82cdc-a5d8-4f27-9ed5-c2c7f14db07a/image.png)

모델 크기가 클수록 성능 개선 관찰, 주관적 품질(OVL)은 1.5B 모델에서 최적, 큰 모델(3.3B)은 텍스트 입력을 더 잘 이해.