---
title: 회귀 분석 - (2) 확률론적 모형
date: 2018-06-19
tags: Regression
category: DataScience
mathjax: true
---
이전에 살펴본 결정론적 모형은 계산한 가중치의 신뢰도를 **부트스트래핑(bootstrapping)** 이라는 방법으로 구해야 한다. 반면, 확률론적 모형에서는 부트스트래핑 없이도 회귀분석 결과의 통계적 특성을 구할 수 있다. 확률론적 선형회귀 모형에서는 데이터가 확률 변수로부터 생성된 표본이라고 가정하기 때문이다.

## 부트스트래핑
회귀 분석에 사용한 데이터가 달라진다면 분석의 결과가 어느 정도 영향을 받는지 알기 위한 **샘플링** 방법이다.


분석에 사용한 데이터가 생성된 표본이거나 모집단에서 선택한 표본이라 가정한다면, 회귀 분석의 결과는 분석에 사용한 표본에 의존적일 것이다. 따라서 다른 표본 데이터를 가지고 분석을 한다면 결과가 달라질 것이다.

> 전체 데이터(절대 모을 수 없는)의 부분만 가지고 분석을 하기 때문에, 부분에 편향된(의존적인) 결과가 나올 수 밖에 없다.


추가 데이터를 가지고 하는 것이 가장 좋지만, 현실적인 어려움 때문에 부트스트래핑 방법에서는 기존 데이터를 재표본화(re-sampling)한다. 재표본화는 기존 N개의 데이터 중에서 **중복선택을 포함하여** 다시 N개의 데이터를 선택한다.


# 확률론적 모형
확률론적 선형회귀 모형에서는 데이터가 확률 변수로부터 생성된 표본이라고 가정하기 때문에, 부트스트래핑 없이도 회귀분석 결과의 통계적 특성을 구할 수 있다.

## 1. 오차의 분포에 대한 가정
오차는 분석의 noise라고 생각하면 된다.

### 1-1. 선형 정규 분포 가정
종속 변수 $y$가 독립 변수 $x$의 선형 조합으로 결정되는 기댓값과 고정된 분산 $\sigma^2$을 가지는 정규분포이다.


  $$ y \sim \mathcal{N}(w^Tx, \sigma^2) $$


$y$의 확률 밀도 함수, 여기서 $\theta = (w, \sigma^2) $ 이다. 즉, $x$와 $\theta$에 의해 모양이 변한다.


  $$ p(y \mid x, \theta) = \mathcal{N}(y \mid w^Tx, \sigma^2 ) $$


>$y$가 $x$에 대해 조건부로 정규 분포를 이룬다. (그러나 $x, y$가 무조건부 정규분포 일 필요는 없다.)


이 관계식을 오차(disturbance) $\epsilon_i$ 개념으로 변환하면 다음과 같이 표현할 수 있다.


  $$\epsilon_i = y - w^Tx$$


오차의 확률 밀도 함수는 다음과 같다. (기댓값 0, 분산 $\sigma^2$)


  $$p(\epsilon \mid \theta) = \mathcal{N}(0, \sigma^2) $$



### 1-2. 외생성 가정

오차($\epsilon$)의 기댓값은 **독립 변수 $x$에 상관없이** 항상 0이라 가정


$$\text{E}[\epsilon \mid x] = 0$$


위 가정 + 전체기댓값의 법칙-> 오차 $\epsilon$의 무조건부 기댓값도 0


$$\text{E}[\text{E}[\epsilon \mid x]] = \text{E}[\epsilon] = 0$$


오차 $\epsilon$과 독립 변수 $x$는 무상관


$$\text{E}[\epsilon x] = 0$$


### 1-3. 조건부 독립 가정


$i$번째 샘플의 오차 $\epsilon_i$와  $j$번째 샘플의 오차 $\epsilon_j$의 공분산 값이 $x$와 상관없이 항상 0이라고 가정


$$\text{Cov}[\epsilon_i, \epsilon_j \mid x] = 0 \;\; (i,j=1,2,\ldots,N)$$

> 샘플의 오차끼리 서로 독립이라는 가정과 같다.

1-2와 1-3의 가정을 통해

$$ \text{E}[\epsilon_i \epsilon_j] = 0 \;\; (i,j=1,2,\ldots,N) $$

위 식을 증명할 수 있고,


$$\text{Cov}[\epsilon] = \text{E}[\epsilon^{} \epsilon^T] = \sigma^2 I$$


공분산 행렬 형태로 표현하면 위 식과 같다.


### 2. 독립 변수에 대한 가정
독립 변수의 공분산 행렬을 full rank 이어야 한다. 즉, 독립 변수에서 서로 독립인 성분이 독립 변수의 갯수만큼 존재해야 한다. 공분산 행렬의 조건이 나쁜 경우, 혹은 full rank가 아닌 경우 올바른 모수 추정이 되지 않는다.


## MLE를 사용한 선형 회귀 분석
위의 가정과 Maximum Likelihood Estimation을 사용하여 가중치 벡터 $w$의 값을 구할 수 있다.


Likelihood

$$\begin{eqnarray}
p(y_{1:N} \,\big|\, x_{1:N}, \theta)
&=& \prod_{i=1}^N \mathcal{N}(y_i \,\big|\, w^T x_i , \sigma^2) \\
&=& \prod_{i=1}^N \frac{1}{\sqrt{2\pi\sigma^2}}\exp\left\{-\frac{(y_i-w^T x_i)^2}{2\sigma^2} \right\}  \\
\end{eqnarray}$$


위 식에 Log를 씌워 log-likelihood를 구한다.


$$
\begin{eqnarray}
\text{LL}  
&=& \log p(y_{1:N} \,\big|\, x_{1:N}, \theta) \\
&=& \log \prod_{i=1}^N \frac{1}{\sqrt{2\pi\sigma^2}}\exp\left\{-\frac{(y_i-w^T x_i)^2}{2\sigma^2} \right\}  \\
&=& -\dfrac{1}{2\sigma^2} \sum_{i=1}^N (y_i-w^T x_i)^2 - \dfrac{N}{2} \log{2\pi}{\sigma^2}  \\
\end{eqnarray} $$


행렬로 표시하면,


$$\text{LL}  =  -C_1 (y - Xw)^T(y-Xw) - C_0 = -C_1(w^TX^TXw -2 y^TXw + y^Ty) - C_0$$

$$C_1 =  -\dfrac{1}{2\sigma^2}$$


$$C_0 =  \dfrac{N}{2} \log{2\pi}{\sigma^2}$$


최적화하면 결정론적 방법에서의 OLS와 동일한 결과를 얻을 수 있다.


## 잔차의 분포
또한, 지금까지 해온 선형 회귀모형에 따르면 잔차 $\epsilon = y - \hat{w}^Tx $ 도 정규 분포를 따른다.


오차 $\epsilon$과 잔차 $e$의 관계를 나타내자면,


$$\hat{y} = X\hat{w} = X (X^TX)^{-1}X^T y = Hy$$


위 $H$ 행렬은 $HH = H$라는 성질을 가진 대칭행렬이다. projection 혹은 influence 행렬이라고 부른다.


$$e = y - \hat{y}= y - Hy = (I - H) y$$


$M = I - H$라고 정의한다. 이 행렬 $M$은 잔차(residual)행렬 혹은 rejection행렬이라고 부른다. $MM=M$이라는 성질을 가진다. 위 식을 M을 이용해 바꾸면, 다음과 같은 식이 된다.


$$e = My = M (Xw + \epsilon) = MXw + M\epsilon$$


여기서,


$$X^TM = X^T(I - H) = X^T - X^TH = X^T - X^TX (X^TX)^{-1}X^T = 0$$


위의 결과에서 $M$은 대칭이라서 $M^TX = MX = 0$ 이고, 처음 잔차식에 값을 대입하면


$$e = MXw + M\epsilon = M\epsilon$$


위와 같은 결과가 나오는데, 이것은 잔차 $e$가 오차 $\epsilon$의 선형 변환이라는 의미다. 그리고 정규분포의 선형 변환은 마찬가지로 정규분포하기 때문에, 잔차도 정규 분포를 따른다.

> - 잔차 : 내 예측값과 실제값의 차이
> - 오차 : 모델에 포함된 노이즈


---


### References

[데이터 사이언스 스쿨 - 확률론적 선형회귀모형](https://datascienceschool.net/view-notebook/743cdedec523447a907b2b0abda45533/)
