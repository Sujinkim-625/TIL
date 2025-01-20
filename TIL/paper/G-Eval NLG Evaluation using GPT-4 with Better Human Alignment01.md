 [G-Eval: NLG Evaluation using GPT-4 with Better Human Alignment](https://arxiv.org/abs/2303.16634)

### Abstract
이 논문은 자연어 생성 시스템의 품질 평가에 대한 문제를 다루며, 특히 기존 평가 지표의 한계와 대안을 제시한다.

**1. 배경 및 문제제기**
- 기존 지표의 한계: BLEU와 ROUGE 같은 기존 reference-based 지표는 창의성과 다양성이 요구되는 작업에서 인간 평가와의 상관성이 낮다.
- 새로운 접근법: reference-free 평가 방법으로 LLMs를 사용하는 것을 제안한다.

**2. G-EVAL**
- CoT: LLMs의 사고 과정을 단계적으로 표현하여 더 정교한 평가를 수행한다.
- Form-Filling: 평가 기준을 구조화된 형식으로 채워넣는 방식으로, 더 명확하고 체계적인 평가를 가능하게 한다.

**3. 의의와 결론**
- G-EVAL은 기존의 평가 방식을 능가하며, 대화나 창의성을 요구하는 task에서 효과가 뛰어나다.
- 텍스트 요약과 대화 생성 태스크에 있어 GPT-4를 백본으로 사용하는 G-EVAL 프레임워크는 사람 평가와의 피어슨 상관계수가 0.514로, 기존에 제안된 모든 방법을 크게 능가하는 상관성을 보인다. 
- 단, 이 방식은 프롬프트에 민감하기 때문에 CoT를 통해 더 많은 컨텍스트와 가이드를 제공할 때 alignment가 높아진다.
- LLM 기반의 평가를 면밀히 조사한 결과, 사람이 작성한 텍스트보다 LLM이 작성한 텍스트를 선호하는 편향성을 발견하였다. 따라서 LLM 기반의 매트릭을 LLM 학습의 보상 신호로 사용할 경우, 편향된 결과를 야기할 수 있다. (self reinforcement of LLMs)

### Method: G-EVAL Framework
1.생성 결과물 평가를 위한 프롬프트 설계
프롬프트: 평가하려는 태스트에 대한 정의와 원하는 평가 기준을 정의하는 자연어 명령어

텍스트 요약의 경우 아래와 같은 프롬프트를 사용할 수 있다.
```
You will be given one summary written for a news article. Your task is to rate the summary on one metric. Please make sure you read and understand these instructions carefully. Please keep this document open while reviewing, and refer to it as needed.
뉴스 기사에 대해 작성된 하나의 요약이 주어질 것이다. 너의 임무는 하나의 매트릭으로 요약문을 평가하는 것이다.
아래 지침을 주의 깊게 읽고 이해하도록 하라. 검토를 진행하는 동안 이 지침을 열어두고, 필요할 때 참조하라.
```
태스크에 대한 평가에 필요한 기준을 프롬프트화하면 된다.
평가 기준은 일관성, 간결성, 문법적 오류 없음 등 태스크에 따라 다양하게 설정할 수 있다.

텍스트 요약에서 일관성 항목을 평가하고자 하는 경우, 아래와 같은 내용을 프롬프트에 추가할 수 있다.
```
Evaluation Criteria:
Coherence (1-5) - the collective quality of all sentences. We align this dimension with the DUC quality question of structure and coherence whereby ”the summary should be well-structured and well-organized. The summary should not just be a heap of related information, but should build from sentence to sentence to a coherent body of information about a topic.”

평가 기준:
일관성 (1-5) - 일관성은 모든 문장에 대한 총체적인 품질을 의미한다. 이 기준은 다음과 같은 구조와 일관성에 대한 DUC 품질 질문과 관련이 있다: "요약문은 잘 구조화되고 잘 정리되어 있어야 한다. 요약문은 관련된 정보를 나열한 수준이 아니라 하나의 주제에 대한 일관된 정보로 문장에서 문장으로 이어져 있어야 한다."
```

2.CoT
CoT는 텍스트 생성 과정에서 LLM이 생성하는 중간 representation 시퀀스이다.
생성된 텍스트를 평가하는 단계에서는 단순한 정의 이상의 자세한 평가 지침이 필요한데, 각각의 태스크에 대해 이러한 평가 단계를 수동으로 설계하는 것은 많은 시간이 소요된다. 
대규모 언어모델은 이러한 평가단계를 스스로 생성할 수 있기 때문에 CoT를 통해 LLM이 텍스트를 평가할 수 있도록 더 많은 컨텍스트와 지침을 제공할 수 있고, 평가 과정과 결과를 설명하는데 도움이 될 수 있다.

텍스트 요약에서 일관성 항목을 평가하고자 하는 경우, 프롬프트에 "Evaluation Steps:"라는 표현을 추가하여 LLM이 생성하도록 한 CoT이다.
```
1. Read the news article carefully and identify the main topic and key points.
2. Read the summary and compare it to the news article. Check if the summary covers the main topic and key points of the news article, and if it presents them in a clear and logical order.
3. Assign a score for coherence on a scale of 1 to 5, where 1 is the lowest and 5 is the highest based on the Evaluation Criteria.

1. 뉴스 기사를 주의 깊게 읽고 주요 주제와 요점을 한다.
2. 요약문을 읽고 뉴스 기사와 비교한다. 요약이 뉴스 기사의 주요 주제와 요점을 명확하고 논리적인 순서로 제시하였는지 확인한다.
3. 일관성에 대한 점수를 1-5점까지의 척도로 부여한다. 이때 평가 기준에 따라 1은 가장 낮은 점수, 5가 가장 높은 점수이다.
```

3. Scoring Function
아래의 세 가지를 input으로 LLM을 호출한다
```
생성 결과물 평가를 위한 프롬프트
자동으로 생성된 CoT
입력 컨텍스트와 평가해야할 대상 텍스트
```

G-EVAL은 양식 채우기 방식으로 평가 작업을 직접 수행한다. 예를 들어, 텍스트 요약의 일관성을 평가하기 위해 텍스트 요약에서 프롬프트와, CoT, 뉴스 기사, 요약문을 입력으로 LLM을 호출하여 각 평가 항목에 대해 1에서 5까지의 점수를 출력한다. 

하지만, 이렇게 직접 점수를 출력하는 방식에는 두가지 문제가 있다:
- 일부 평가 태스크에서는 하나의 숫자가 점수 분포를 지배한다. 예를 들어 1-5점 척도에서는 3점이 지배적으로 출력되었다. 이로 인해 사람이 판단하기에 점수가 낮은 결과물과 G-EVAL 점수와의 상관관계가 낮아질 수 있다. 
- 프롬프트에서 소수점 단위의 평가를 명시적으로 요청하더라도 LLM은 주로 정수 아웃풋을 출력하였다. 이로 인해 평가에서 동점이 많이 발생하였고 이로 인해 생성된 텍스트 간의 미묘한 차이를 포착하지 못한다.

이를 해결하기 위해 LLM의 출력 토큰 확률을 사용하여 점수를 정규화하고 그들의 가중합을 최종 결과로 사용할 것을 제안한다. 이 방식을 통해 생성된 텍스트와 품질과 다양성을 더 반영할 수 있는 연속적인 점수를 얻을 수 있다.
![](https://velog.velcdn.com/images/s0o0_jiiin/post/e50ff57f-1e85-4527-bac0-b1500ede6bcd/image.png)

### G-EVAL은 LLM이 생성한 텍스트를 선호하는가
LLM을 평가자로 사용할 때 우려사항 중 하나는 모델이 직접 작성한 고품질 텍스트보다 LLM 자체에서 생성한 결과물을 더 선호할 수 있다는 점이다.

데이터는 다음의 세 카테고리로 나누어진다:
1) 사람이 평가했을 때 GPT-3.5가 작성한 요약보다 사람이 작성한 요약이 더 높은 점수를 받은 경우
2) 사람이 평가했을 때  GPT-3.5가 작성한 요약보다 사람이 작성한 요약이 더 낮은 점수를 받은 경우
3) 사람이 평가했을 때 GPT-3.5가 작성한 것과 사람이 작성한 요약이 같은 품질이라고 평가한 경우 

![](https://velog.velcdn.com/images/s0o0_jiiin/post/3ddd70ae-9e9b-4079-9826-1cba377a1eb7/image.png)

그림에서 볼 수 있듯이 G-EVAL-4는 항상 GPT-3.5가 작성한 요약을 선호한다.

그 이유는 다음과 같다.
- NLG 평가의 본질적 어려움: 고품질 시스템의 출력(NLG 텍스트)은 평가가 본질적으로 어렵다. 인간 평가자 간 합의도(Inter-Annotator Agreement)가 매우 낮았으며, 크리펜도르프 알파(Krippendorff’s alpha)는 0.07로 나타남.
- LLM 평가자의 LLM 출력 편향: G-EVAL-4는 LLM 생성 요약에 대해 편향을 가질 가능성이 있음. 이는 평가 모델(G-EVAL)과 생성 모델(GPT-3.5)이 평가 기준을 공유할 가능성 때문.LLM은 텍스트를 생성하고 평가할 때 동일한 개념과 기준을 사용할 수 있으며, 이로 인해 LLM 생성 텍스트에 더 높은 점수를 부여할 가능성이 있음.




참고 [논문리뷰 G-Eval: LLM을 사용해 인간의 견해와 보다 일치하는 NLG 평가 시스템 구축하기](https://littlefoxdiary.tistory.com/123?utm_source=chatgpt.com)