---
title: Chain of Thought
date: 2025-04-06 20:00:00 +0900
categories: [생성형 AI, 만들기]
tags: [llm, prompt]     # TAG names should always be lowercase
---

# 추론 모델의 시조 : Chain of Thought
![Banner](/assets/img/2025-04-06-chain-of-thought-01-banner.png)

최근 AI 모델들이 점점 더 똑똑해지고 있습니다. OpenAI의 o3 시리즈, Deepseek R1, Claude 3.7, Gemini 2.5 등 최신 모델들은 단순히 텍스트를 생성하는 것을 넘어 '사고 과정'을 갖추고 있습니다. 이런 모델들의 핵심 특징은 바로 'thinking' 능력, 즉 문제를 풀기 위해 단계적으로 추론하는 능력이죠.

## 🧠 OpenAI 추론 모델 가격 비교 (2025년 4월 기준)

| 모델명       | 입력 비용 (1M 토큰) | 출력 비용 (1M 토큰) | 비고                |
| ------------ | ------------------- | ------------------- | ------------------- |
| o3           | $10.00              | $40.00              | 최신 고급 추론 모델 |
| GPT-4.1      | $ 2.00              | $ 8.00              | 최신 고성능 모델    |
| o4-mini      | $ 1.10              | $ 4.40              | 최신 경량 추론 모델 |
| GPT-4.1 mini | $ 0.40              | $ 1.60              | 최신 경량 모델      |

보시다시피 추론에 특화된 모델들은 일반 모델보다 훨씬 비싼 편입니다. 그만큼 복잡한 문제 해결 능력이 뛰어나기 때문이죠. 하지만 모든 상황에서 이런 고가 모델을 사용하기는 어렵습니다.
그렇다면 우리가 궁금해야 할 것은 이것입니다: 왜 'thinking'을 통한 추론 과정이 AI의 성능을 크게 향상시킬까요? 그리고 비싼 모델을 사용하지 않고도 이런 추론 능력을 활용할 방법은 없을까요? 바로 여기서 Chain of Thought(CoT) 프롬프팅이 중요해집니다.

### 이 글에서 다룰 내용

이 가이드에서는 LLM과 프롬프트 엔지니어링에 관심 있는 개발자분들을 위해 다음 내용을 상세히 다룹니다:

1. CoT 프롬프팅의 핵심
2. 실제 구현 시 주의할 점
3. 실제 적용 사례 연구

특히 dspy 라이브러리를 활용한 실제 구현 예제와 성능 비교 실험을 통해, 여러분의 AI 프로젝트에 바로 적용할 수 있는 실용적인 가이드를 제공합니다.

## Chain of Thought (CoT) 프롬프팅이란?

### 핵심 개념

CoT 프롬프팅은 LLM에게 단순히 최종 답변만 요구하는 것이 아닙니다. 대신, **답변에 도달하기까지의 중간 추론 과정을 단계별로 생각하고 출력하도록 유도하는 기법**입니다.

![표준 프롬프팅 vs CoT 프롬프팅 비교](/assets/img/2025-04-06-chain-of-thought-02-comparison.png)
*표준 프롬프팅과 CoT 프롬프팅의 결과 비교 - 같은 답변도 추론 과정이 다름*

혹시 여러분은 누군가에게 지식을 설명하면서 오히려 자신의 이해가 더 깊어진 경험이 있으신가요? 흔히 '가르치면서 배운다'는 말처럼, 설명을 통해 스스로의 생각을 정리하는 효과가 있습니다. 수학 문제를 풀 때도 마찬가지입니다. 정답을 바로 도출하기보다는 풀이 과정을 단계별로 적어 내려가면서 사고가 전개되는 것을 느낄 수 있습니다. Chain of Thought(CoT) 프롬프팅의 핵심 원리도 이와 같습니다. LLM이 스스로 추론 과정을 거치도록 유도함으로써, 마치 사람이 생각하는 것처럼 문제 해결 능력을 향상시키는 것이죠.

## ️ CoT 프롬프팅 구현 방법

### 기본 구현 단계

이 챕터에서는 CoT 프롬프팅의 기본 구현 단계를 살펴봅니다. 최신 LLM은 'thinking' 기능을 내장하고 있어 CoT를 더 쉽게 활용할 수 있습니다.

최근 OpenAI o1, o3 시리즈, Deepseek R1, Claude 3.7, Gemini 2.5 등 최신 모델들은 모두 'thinking' 기능을 포함하고 있습니다. Chain of Thought를 사용하는 가장 쉬운 방법은 바로 이러한 추론 능력을 지닌 고급 모델을 쓰는 것입니다. 프롬프트에 '단계적 절차'를 지시할 필요도 없고 성능도 뛰어납니다.

하지만 이러한 고급 모델은 비용이 매우 비싸다는 단점이 있습니다. 따라서 행의 갯수가 많은 데이터에서 데이터를 추출하거나, 분류를 하는 경우에는 이러한 비싼 모델을 사용하기 어렵습니다.

이러한 경우, 프롬프트를 잘 설계하면 비용 효율적으로 높은 결과를 기대할 수 있습니다.

1. **Few-shot 예제 활용**: LLM에게 CoT 프롬프팅을 사용하는 방법을 보여주는 몇 가지 예제를 제공하여, LLM이 CoT 프롬프팅의 작동 방식을 학습하도록 돕습니다. Few-shot 예제는 LLM이 새로운 문제에 대해 CoT 프롬프팅을 적용하는 데 필요한 지침을 제공합니다.
   ```python
   prompt = """
   문제: 15개의 사탕이 들어있는 상자가 3개 있습니다. 각 상자에는 5개의 파란색 사탕과 10개의 빨간색 사탕이 들어 있습니다. 빨간색 사탕은 몇 개입니까?
   
   단계별로 생각해봅시다:
   1) 먼저, 각 상자에 들어있는 빨간색 사탕의 수를 파악합니다: 10개
   2) 다음으로, 상자의 수를 파악합니다: 3개
   3) 마지막으로, 빨간색 사탕의 총 개수를 계산합니다: 10개 * 3 = 30개
   
   최종 답변: 30개
   
   문제: [새로운 문제]
   ```

   - **팁**: 다양한 유형의 문제에 대한 예제를 제공하면 LLM이 CoT 프롬프팅을 더욱 General하게 적용할 수 있습니다. 예제를 직접 만드는 것이 어려우면, LLM에게 예제를 만들어 달라고 요청하세요.

2. **Zero-shot CoT 활용**: Few-shot 예제 없이 LLM에게 직접 CoT 프롬프팅을 적용하여, LLM의 자체적인 추론 능력을 활용합니다. Zero-shot CoT는 LLM이 사전 학습된 지식을 바탕으로 추론 과정을 생성하도록 유도합니다.
   ```python
   prompt = """
   문제: [구체적인 문제 기술]
   
   단계별로 생각해봅시다.
   """
   ```
   - **팁**: LLM의 크기가 클수록 Zero-shot CoT의 효과가 더욱 좋습니다. 

### 구조화된 답변의 중요성

LLM에게 질문할 때, 단순히 답을 얻는 것 이상으로 **답변의 구조**가 중요합니다. 구조화된 답변이란, LLM이 답변을 생성하는 과정을 명확하게 제시하고, 각 단계별 추론 과정을 논리적으로 연결하여 최종 결론에 도달하는 방식을 의미합니다. 이는 마치 사람이 복잡한 문제를 해결할 때 단계별로 사고하는 과정과 유사합니다.

구조화된 답변은 다음과 같은 이점을 제공합니다.

*   **정확도 향상**: LLM이 추론 과정을 체계적으로 수행하도록 유도하여, 오류 발생 가능성을 줄이고 정확도를 높입니다.
*   **투명성 증대**: 답변 과정을 명확하게 제시함으로써, 사용자가 LLM의 사고 과정을 이해하고 결과의 신뢰성을 평가할 수 있도록 돕습니다.
*   **디버깅 용이**: 오류 발생 시, 추론 과정을 역추적하여 문제의 원인을 쉽게 파악하고 해결할 수 있습니다.

다음은 DSPY 라이브러리를 사용하여 구조화된 답변을 구현하는 예제입니다.

```python
class QA_cot(dspy.Signature):
    """
    다음 질문에 대해 추론과정과 함께 답하세요.
    단계별로 생각해봅시다.
    """
    query: str = dspy.InputField()
    reasoning: str = dspy.OutputField()
    answer: str = dspy.OutputField()

query = "어떤 자동차가 1시간에 60km를 달립니다. 2시간 30분 동안 달리면 총 몇 km를 갈 수 있을까요?"
cot_good = dspy.Predict(QA_cot)
```

위 코드에서 `reasoning: str = dspy.OutputField()`는 LLM이 추론 과정을 출력하도록 지시하는 중요한 부분입니다. `answer` 뿐만 아니라 `reasoning`을 통해 **사고의 흔적**을 함께 보여주도록 유도하는 것이죠. 이렇게 함으로써 LLM은 문제 해결 과정을 더욱 명확하게 제시하고, 결과적으로 정답률을 높일 수 있습니다.

![CoT가 잘 적용된 예 : 정답율 70%](/assets/img/2025-04-06-chain-of-thought-03-good-example.png)

## 흔한 실수와 잘못된 사용 패턴

### 1. 생각 토큰 미생성: 추론 과정 누락의 함정

CoT 프롬프팅을 사용했음에도 불구하고 답변 스키마에 추론 과정을 포함하지 않는 경우, CoT는 효과적으로 작동하지 않습니다.

```python
class QA(dspy.Signature):
    """
    다음 질문에 답하세요.
    단계별로 생각해봅시다.
    """
    query: str = dspy.InputField()
    answer: str = dspy.OutputField()
```

위 예제는 앞선 실전 예제와 달리 `reasoning` 필드가 생략되었습니다. LLM은 단순히 다음에 생성할 단어를 예측하는 모델일 뿐, 실제로 사고하는 능력이 없습니다. 따라서 "단계별로 생각해보라"고 지시하더라도, 추론 내용을 출력하도록 명시하지 않으면 CoT는 제대로 기능하지 않습니다. 

![CoT가 잘못 적용된 예 : 정답율 30%](/assets/img/2025-04-06-chain-of-thought-04-bad-example.png)

### 2. 답변 후 추론: 논리적 일관성 붕괴

답변을 먼저 제시하고 추론 과정을 나중에 설명하는 경우에도 문제가 발생합니다.

```python
class QA_order(dspy.Signature):
    """
    질문에 대한 답변하세요.
    단계별로 생각해봅시다.
    """
    query: str = dspy.InputField()
    answer: str = dspy.OutputField()
    reasoning: str = dspy.OutputField()
```

이번에는 답변 항목에 `reasoning`을 추가했지만, 순서가 잘못되었습니다. LLM은 답변을 먼저 생성한 후 추론을 덧붙이게 되므로, 추론이 답변에 실질적인 영향을 미치지 못합니다. 심지어 LLM은 이미 내놓은 답변을 정당화하는 방향으로 추론을 왜곡할 위험도 있습니다. 이는 마치 결론을 미리 정해놓고 짜맞추기식 근거를 제시하는 것과 같습니다.

## 실제 적용 사례 연구

### 1. 수학 문제 해결 성과

CoT 프롬프팅은 수학 문제 해결 분야에서 괄목할 만한 성과를 보여주고 있습니다. 특히, GSM8K 데이터셋을 활용한 실험에서 CoT 적용 전에는 20%에 불과했던 정확도가 CoT 적용 후에는 70%까지 상승하는 결과를 얻었습니다.

- **핵심 가치**: CoT 프롬프팅은 복잡한 수학 문제 해결 능력을 향상시키는 데 효과적입니다.
- **제공하는 인사이트**: CoT는 다단계 계산 문제에서 특히 뛰어난 성능을 발휘합니다.
- **차별화 포인트**: CoT는 문제 해결 과정을 명확하게 제시하여, 사용자가 결과의 신뢰성을 검증하고 디버깅하는 데 도움을 줍니다.

### 2. 상식 추론 능력 향상

CoT 프롬프팅은 LLM의 상식 추론 능력 향상에도 기여합니다. 일반 상식 문제에 대해 단순 답변만 제시했을 때는 65%의 정확도를 보였지만, CoT를 적용했을 때는 80%까지 정확도가 상승했습니다.

- **핵심 가치**: CoT 프롬프팅은 LLM이 상식적인 추론을 수행하는 데 도움을 줍니다.
- **제공하는 인사이트**: CoT는 오답의 원인을 파악하는 데 유용하며, 중간 과정에서 오류를 발견할 가능성이 높습니다 (오답의 90%는 중간 과정에서 오류 발견 가능).
- **차별화 포인트**: CoT는 LLM이 문제에 대한 깊이 있는 이해를 바탕으로 답변하도록 유도합니다.

### 3. 모델 크기별 성능 비교

CoT 프롬프팅의 효과는 모델의 크기에 따라 다르게 나타납니다. 일반적으로 모델 크기가 클수록 CoT 적용 시 성능 향상 폭이 커지는 경향이 있습니다.

| 모델 크기 | CoT 적용 시 향상도 |
| --------- | ------------------ |
| 7B        | +5%                |
| 13B       | +12%               |
| 100B+     | +25% 이상          |

- **핵심 가치**: CoT 프롬프팅은 모델의 크기와 관계없이 성능 향상에 기여합니다.
- **제공하는 인사이트**: CoT는 대형 모델에서 더욱 큰 효과를 발휘합니다.
- **차별화 포인트**: CoT는 모델의 잠재력을 최대한으로 끌어올리는 데 도움을 줍니다.

## 결론

CoT 프롬프팅은 LLM의 추론 능력을 획기적으로 향상시키는 강력한 도구임이 분명합니다. 이 기법을 통해 우리는 다음과 같은 놀라운 결과들을 얻을 수 있습니다:

1. **정확성 향상**: 복잡한 문제에 대한 해결 능력이 강화되어, 이전에는 풀 수 없었던 문제들도 해결할 수 있게 되었습니다. 또한, 오류 발생률이 대폭 감소하여 결과의 신뢰성이 높아졌습니다.
2. **투명성 확보**: LLM의 추론 과정을 명확하게 이해할 수 있게 되어, 디버깅이 용이해지고 결과에 대한 설명 가능성이 향상되었습니다. 이는 LLM을 더욱 신뢰하고 효과적으로 활용할 수 있는 기반을 마련해줍니다.
3. **실용성 증대**: CoT 프롬프팅은 다양한 분야에 적용 가능하며, 구현이 비교적 간단하고 즉각적인 성능 개선 효과를 얻을 수 있습니다. 이는 CoT 프롬프팅이 LLM 개발 및 활용에 있어서 매우 실용적인 도구임을 의미합니다.

앞으로 AI 기술이 더욱 발전함에 따라, CoT 프롬프팅은 LLM의 잠재력을 최대한으로 발휘하는 데 더욱 중요한 역할을 할 것으로 기대됩니다. CoT 프롬프팅을 통해 우리는 LLM을 더욱 똑똑하고 신뢰할 수 있는 동반자로 만들 수 있을 것입니다.

## 📚 참고 자료


### 핵심 논문
1. Wei et al. (2022). *Chain-of-Thought Prompting Elicits Reasoning in Large Language Models.*
2. Kojima et al. (2022). *Large Language Models are Zero-Shot Reasoners.*
3. Zhang et al. (2022). *Automatic Chain of Thought Prompting in Large Language Models.*
4. Wang et al. (2022). *Self-Consistency Improves Chain of Thought Reasoning in Language Models.*

### 웹 리소스
- [Prompting Guide](https://www.promptingguide.ai/techniques/cot)
- [Mercity AI Blog](https://www.mercity.ai/blog-post/guide-to-chain-of-thought-prompting)
- [Learn Prompting](https://learnprompting.org/docs/intermediate/chain_of_thought)
- [PromptHub Blog](https://www.prompthub.us/blog/chain-of-thought-prompting-guide)
- [Cohere Docs](https://docs.cohere.com/v2/docs/advanced-prompt-engineering-techniques)
