---
title: 다중 분류 문제 성능 평가 방법 이해하기 [기본편]
date: 2020-04-12
tags: MachineLearning
category: datascience
toc: False
mathjax: true
header:
  teaser: /assets/img/post/data/william-warby-WahfNoqbYnM-unsplash.jpg
---

어떤 모델, 혹은 방법을 쓰던 분류 문제는 그 의도에 따라 다양한 성능평가 방식을 사용합니다. 사람, 고양이, 개 3개의 클래스를 분류하는 다중 분류(multi label) 예제를 통해 정리해보겠습니다. 여기에서는 가장 기본이 되는 Accuracy, Recall, Precision, F-score와 Fall-out에 대해 정리합니다.

![img](/assets/img/post/data/william-warby-WahfNoqbYnM-unsplash.jpg)
*Photo by  [William Warby](https://unsplash.com/@wwarby?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)  on  [Unsplash](https://unsplash.com/s/photos/measure?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)*

> 분류 문제에서 분류해야되는 각 종류를 클래스라고 합니다. (Class를 Classfication!)  
> 카테고리, 라벨, 종속변수 등으로도 불립니다.

## Intro

일반적인 이진분류 문제에서는 다음과 같은 분류 결과 표를 만들 수 있습니다. 불량 검출의 예에서는 불량품을 검출하는 것이 목표니까, 주로 Positive를 불량으로 둡니다.

|실제 \ 예측 | 불량으로 예측 | 정상으로 예측 |
|:--:|:--:|:--:|
|불량|**True** *Positive* | False Negative|
|정상|False *Positive*| **True** Negative |

클래스가 2개인 이진분류 문제에서는 위처럼 총 4가지 경우가 있습니다. 불량을 불량(**True** *Positive*)이라 하거나 정상(False Negative)이라 할 수 있고, 또 정상을 불량(False *Positive*)이라 하거나 정상(**True** Negative)이라 할 수 있죠. **True는** 정답, False는 오답. *Positive*는 불량으로 예측, Negative는 정상으로 예측.

클래스가 3개인 경우에는 아래처럼 표현할 수 있습니다.

|실제 \ 예측 | 사람으로 예측 | 고양이로 예측 | 개로 예측 |
|:--:|:--:|:--:|:--:|
|사람  |**True** Person|False Cat|False Dog|
|고양이|False Person|**True** Cat|False Dog|
|개   |False Person|False Cat|**True** Dog|

사람을 사람이라고, 고양이를 고양이라고, 개를 개라고 하면 True입니다. 나머지는 모두 False입니다. 아래에서 정리할 분류 성능 평가는 모두 이 값의 비율로 계산합니다. 다중 클래스는 OvR(One-vs.-Rest) 문제로 자기 클래스는 Positive, 나머지는 모두 Negative로 하여 계산합니다. *여기서 False 태그는 ~라고 예측했지만, 실제로는 아니다로 생각해주세요. False Person: 사람이라고 예측했지만, 실제로는 아니다.*

## Scores

### Accuracy

정확도는 전체 개수 중 정답을 맞춘 개수 입니다. 문제 10개 중에 8개 맞추면 10점 만점에 8점!

$\text{Accuracy}={\dfrac{\text{정답 클래스 수}}{\text{모든 클래스 수}}}$

> 정확도는 데이터에 따라 주의해서 봐야합니다. 희귀병 유무 검사를 정확도로 하면 어떻게 될까요? 덮어놓고 다 정상이라고 해도 정확도는 99.999...%가 될 겁니다. 그리고 희귀병은 검출되지 못하겠죠. 이처럼 데이터의 불균형이 심한 경우에는 높은 정확도의 의미를 잘 봐야합니다.

### Precision & Recall

정밀도(Precision)는 어떤 클래스라고 예측한 수 중 실제로 그 클래스가 맞은 수를 표현합니다.

|실제 \ 예측 | 사람으로 예측 | 고양이로 예측 | 개로 예측 |
|:--:|:--:|:--:|:--:|
|사람  |True Person|<span style="color:red">False Cat|False Dog|
|고양이|False Person|<span style="color:green">True Cat|False Dog|
|개   |False Person|<span style="color:red">False Cat|True Dog|

고양이 클래스의 정밀도를 볼 때는 고양이라고 **예측한 열**을 중심으로 보면 됩니다. 예를 들어, 내가 고양이라고 한 10마리 중 실제로 고양이가 8마리가 있으면 10점 만점에 8점!

$\text{Precision}_{cat}={\dfrac{\text{고양이라고 예측한 개체 중 실제 고양이 수}}{\text{고양이라고 예측한 수}}}$

반면, 재현율(Recall)은 실제 클래스의 개수 중 얼마나 맞혔는지를 표현합니다.

|실제 \ 예측 | 사람으로 예측 | 고양이로 예측 | 개로 예측 |
|:--:|:--:|:--:|:--:|
|사람  |True Person|False Cat|False Dog|
|고양이|<span style="color:red">False Person</span>|<span style="color:green">True Cat</span>|<span style="color:red">False Dog</span>|
|개   |False Person|False Cat|True Dog|

고양이 클래스의 재현율을 볼 때는 분류 결과표에서 실제 고양이 행을 중심으로 보면 이해가 쉽습니다. 고양이가 10마리 있을 때 내가 9마리를 찾아냈다면 10점 만점에 9점!

$\text{Recall}_{cat}={\dfrac{\text{실제 고양이 중 고양이라고 예측한 수}}{\text{실제 고양이 수}}}$

> **Precision과 Recall**  
> 두 지표는 단일 지표로만 보면 왜곡하기(혹은 오해하기) 쉬운 평가지표입니다. 고양이 클래스의 재현율(Recall)을 1 (100%)로 만들고 싶다면, 전부 다 고양이라고 하면 됩니다. 반면 정밀도(Precision)를 1로 만들고 싶다면, 정말 고양이다 싶은 딱 1개만 고양이라고 하면 됩니다. 고양이가 아니라 범죄자면 검거율이 되겠죠? 전 국민이 감옥에 들어가 있던 범죄자만 모두 감옥에 들어가 있던 재현율은 1로 동일합니다. 반면, 전자의 정밀도는 거의 0에 가깝겠죠.

### F-score

이처럼 정밀도와 재현율은 서로 상충하는 경우가 많습니다. 임계치를 바꾸는 식으로 하나의 값을 내리고 다른 하나를 올릴 수 있기 때문입니다. 따라서 두 값의 가중조화평균을 계산하여 평균적인 성능을 확인할 수 있습니다.

|실제 \ 예측 | 사람으로 예측 | 고양이로 예측 | 개로 예측 |
|:--:|:--:|:--:|:--:|
|사람  |True Person|<span style="color:red">False Cat|False Dog|
|고양이|<span style="color:red">False Person|<span style="color:green">True Cat|<span style="color:red">False Dog|
|개   |False Person|<span style="color:red">False Cat|True Dog|

*분모는 다르지만 분자는 동일합니다.*

$F_\beta = (1 + \beta^2) \, ({\text{precision} \times \text{recall}}) \, / \, ({\beta^2 \, \text{precision} + \text{recall}})$

베타가 1인 경우에는 두 값의 가중치를 동등하게 계산하게 되고, 1 이상이면 정밀도에, 1 미만이라면 재현율에 가중치를 줍니다. 베타가 1인 F-Score를 특별히 F1-Score라고 합니다.

> **왜 조화평균을 사용할까?**  
> 만약 산술평균으로 계산할 경우, 어느 한 쪽이 높은 값을 가지면 다른 한 쪽이 낮은 값을 가지게 되더라도 높은 쪽의 영향을 크게 받으므로 산술평균 값은 비교적 큰 값을 갖기 때문입니다. 즉, 값이 왜곡됩니다. 반면에 조화평균은 어느 한 쪽이 낮으면, 값도 크게 낮아집니다.

```python
pr = 0.8
rc = 0.1
b = 1
print("산술평균: ", (pr + rc) / 2)
print("조화평균: ", (1+b**2) * (pr * rc) / (b**2 * pr + rc))
# 산술평균:  0.45
# 조화평균:  0.1777777777777778
```

### Fall-out

위양성률은 억울한 애들의 비율입니다.

|실제 \ 예측 | 사람으로 예측 | 고양이로 예측 | 개로 예측 |
|:--:|:--:|:--:|:--:|
|사람  |<span style="color:red">True Person|<span style="color:green">False Cat|<span style="color:red">False Dog|
|고양이|False Person|True Cat|False Dog|
|개   |<span style="color:red">False Person|<span style="color:green">False Cat|<span style="color:red">True Dog|

이번에도 고양이 클래스 관점에서 보겠습니다. 고양이가 아닌 클래스(사람과 개) 중에 고양이라고 오해한 애들의 비율입니다. 위 지표들과는 다르게 낮을 수록 좋습니다. 위양성률은 조금 헷갈려서 숫자 예시를 만들어보았습니다.

|실제 \ 예측 | 사람으로 예측 | 고양이로 예측 | 개로 예측 |
|:--:|:--:|:--:|:--:|
|사람 |<span style="color:red">5|<span style="color:green">0|<span style="color:red">0|
|고양이|1|3|1|
|개   |<span style="color:red">1|<span style="color:green">2</span>|<span style="color:red">2|

$\text{Fall-out}_{cat} = {\dfrac{\text{고양이가 아닌데 고양이라고 말한 개체 수}}{\text{고양이가 아닌 개체 수}}} = {\dfrac{\text{억울 2}}{\text{사람 5} + \text{개 5}}}$

억울한 개 두마리가 고양이로 오해를 받아서 고양이 클래스의 위양성률은 0.2로 볼 수 있습니다.

위양성률을 False Positive Rate (FPR)이라고도 하고, $1 - \text{FPR}$ 의 값을 특이도(Specificy)라고 합니다. 재현율(Recall)을 민감도(Sensitivity)라고도 하는데, 주로 의학 분야에서 특이도와 민감도를 많이 사용한다고 합니다. 민감도와 특이도가 높은 진단 키트일 수록 신뢰성이 높다는 식으로 표현하네요.

### python

scikit-learn을 이용하면 위의 분류 평가지표를 간단히 구할 수 있습니다.

```python
from sklearn.metrics import confusion_matrix
from sklearn.metrics import classification_report

y_true = [0, 0, 0, 0, 0, 1, 1, 1, 1, 1, 2, 2, 2, 2, 2]
y_pred = [0, 0, 0, 0, 0, 0, 1, 1, 1, 2, 0, 1, 1, 2, 2]

target_names = ['사람', '고양이', '개']

print(confusion_matrix(y_true, y_pred))
print(classification_report(y_true, y_pred, target_names=target_names))

# [[5 0 0]
#  [1 3 1]
#  [1 2 2]]
#               precision    recall  f1-score   support

#           사람       0.71      1.00      0.83         5
#          고양이       0.60      0.60      0.60         5
#            개       0.67      0.40      0.50         5

#     accuracy                           0.67        15
#    macro avg       0.66      0.67      0.64        15
# weighted avg       0.66      0.67      0.64        15
```

다중 클래스인 경우 클래스마다 평가 점수가 다르게 되는데, scikit-learn은 macro(단순평균)와 weigthed(클래스별 표본 개수로 가중평균) avg로 표현해줍니다.

## Outro

어떤 문제를 풀던 간에 모든 구성원이 동일한 지표를 설정하고 대화하는 것이 중요합니다. 모두가 **성능**에 대해 말하지만 A는 정확도, B는 정밀도, C는 처리속도에 대해 말하고 있을 수도 있습니다. 최대한 쉽게 써보려 했습니다. AUC, ROC, AP도 한번에 쓰려고 했는데 역시 힘드네요. 다음편에 써보도록 하겠습니다.
