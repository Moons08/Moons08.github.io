---
title: 회귀 분석 - (1) 결정론적 모형
date: 2018-06-19
tags: Regression
category: datascience
mathjax: true
---
14년도 계량 경제학 수업을 들을 때 처음 봤지만, 이제야 조금이나마 알 것 같다. 회귀 분석은 데이터가 어떤 특정한 경향을 보인다는 가정 하에 그 경향성을 찾아 설명하는 것이 목적이다.

# 회귀 분석 (Regression analysis)

회귀 분석은 독립 변수와 종속 변수 간의 관계를 찾는 작업이다. 회귀 분석에는 결정론적 모형과 확률론적 모형이 있다. 결정론적 방법을 보면서 기초를 다져보자.

>부동산(집) 시장을 예로 들면, 집의 가격(종속 변수)과 집의 위치, 상태, 규모, 인근 학교의 수, 편의 시설의 유무 (독립 변수들) 간의 관계를 찾는 것이다.

결정론적 모형에서 관계를 찾는다는 것은 독립 변수 $x$에 대응하는 종속 변수 $y$와 비슷한 $\hat{y}$(예측값)을 출력하는 **함수를** 찾는 것이다.


$$ \hat{y} = f(x) \approx y $$


여기서 **선형 회귀 분석은**, 위의 $f(x)$가 선형이라는 가정 하에 분석을 한다.
> 선형이 아니면? 기저함수를 이용해 선형으로 만들어 버리자


$$ \hat{y} = w_0+w_1x_1+w_2x_2+\cdots+ w_nx_n = w_0 + w^Tx$$


위 식에서 $w$들을 함수의 계수(coefficient)이자 선형 회귀 모형(linear regression model)의 모수(parameter)라고 한다.
> $x$들은? 독립변수들. 따라서 각 독립변수가 $w$만큼씩의 영향으로 $\hat{y}$를 구성

## bias aumentation
그런데 상수항이 0이 아닌 회귀분석모형의 경우 **수식을 간단하게 만들기 위해** 상수항을 **독립 변수에** 추가한다. 이를 바이어스 오그멘테이션이라고 한다.
>그럼 상수항이 0인 경우는? 흠..

$$ x_i =
\begin{bmatrix}
x_{i1} \\ x_{i2} \\ \vdots \\ x_{iD}
\end{bmatrix}
\rightarrow
x_{i,a} =
\begin{bmatrix}
1 \\ x_{i1} \\ x_{i2} \\ \vdots \\ x_{iD}
\end{bmatrix} $$


오그멘테이션을 하면 모든 원소가 1인 벡터가 입력 데이터 행렬에 추가된다.
> [ 오그멘테이션(1), 집의 위치($x_{i1}$), 상태($x_{i2}$), $\cdots$, 편의시설유무($x_{iD}$) ]
> 가짜 독립 데이터를 *한 종류* 추가했다고 생각하면 된다.


$$ X =
\begin{bmatrix}
x_{11} & x_{12} & \cdots & x_{1D} \\
x_{21} & x_{22} & \cdots & x_{2D} \\
\vdots & \vdots & \vdots & \vdots \\
x_{N1} & x_{N2} & \cdots & x_{ND} \\
\end{bmatrix}
\rightarrow
X_a =
\begin{bmatrix}
1 & x_{11} & x_{12} & \cdots & x_{1D} \\
1 & x_{21} & x_{22} & \cdots & x_{2D} \\
\vdots & \vdots & \vdots & \vdots & \vdots \\
1 & x_{N1} & x_{N2} & \cdots & x_{ND} \\
\end{bmatrix} $$


상수항이 있는 경우(0이 아닌 경우), 항상 바이어스 오그멘테이션을 하기 때문에 $x_a, w_a$라는 표시가 생략되는 경우가 많다.

## OLS (Ordinary Least Squares)
최소 제곱법. 잔차제곱합(Residual Sum of Squares)를 최소화하는 벡터$w$를 구하는 것이 목표
>잔차 $ \;\;e \; = \; y - \hat{y} $

$$ \hat{y} = Xw $$

$$ e = y - \hat{y} = y - Xw $$

제곱합을 구한다.

$$ \text{RSS}
=  e^Te
= (y - Xw)^T(y - Xw)
= y^Ty - 2y^T X w + w^TX^TXw  
$$


RSS를 최소화하는 $w$를 찾기 위해 미분
- *최저점에서 미분한 값이(기울기가) 0이 되니까*

$$\dfrac{d \text{RSS}}{d w} = -2 X^T y + 2 X^TX w$$

$$X^TX w^{\ast} = X^T y$$

원하던 벡터 $w$를 구할 수 있다.

$$w^{\ast} = (X^TX)^{-1}X^Ty$$

그리고 다음과 같은 식을 정규 방정식(normal equation)이라 한다.

$$ X^T y- X^TX w = 0$$

이걸 인수분해 하면,

$$ X^T (y- X w) = X^Te =0$$


$X^T$와 $e$의 내적이 0이다. 즉, 직교(orthogonal)한다.
- 모든 차원 d(독립 변수 종류)에 대해 $x_d$는 잔차 벡터 $e$와 직교한다.
- 모든 차원이 이루는 어떤 공간에 대해 $y$를 수직으로 projection한 위치에 $\hat{y}$가 존재한다.


---

### References
- [데이터 사이언스 스쿨 - 선형 회귀분석의 기초](https://datascienceschool.net/view-notebook/58269d7f52bd49879965cdc4721da42d/)
