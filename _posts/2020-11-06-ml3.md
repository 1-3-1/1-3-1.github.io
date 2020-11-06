---
layout: post
title: "[ML] 3 Linear Regression"
date: 2020-11-02
categories: ETC
tags: [MahineLearning,Development]
---

## Linear Regression 이란?
선형 회귀는 종속 변수 y 와 한 개 이상의 독립 변수 x와의 선형 상관 관계를 모델링하는 회귀분석 기법.

주어진 x의 값에 대한 예측값을 H(x) 라 하고 그에 대한 식을 다음과 같이 작성한다. 이를 Hypothesis라 한다.


`H(x) = Wx + b`
이것을 얼마나 잘 예측했는지 판단하는 것을 Cost function이라하고 다음과 같이 나타낸다.


`( H(x) - y )^2`
모든 x에 대한 cost function의 합을 줄이는 것이 선형회귀의 목표 라고 할 수 있다.

## TF Operation을 이용한 간단한 Linear Regression 구현

```python
import tensorflow as tf


# x 랑 y 선언
x_train = [1,2,3]
y_train = [1,2,3]

W = tf.Variable(tf.random_normal([1]),name='weight')
b = tf.Variable(tf.random_normal([1]),name='bias')

# hypothesis 정의. W가 1, b가 0이 나와야함.
hypothesis = x_train * W + b

# cost 정의
cost = tf.reduce_mean(tf.square(hypothesis - y_train))

# minimize
optimizer = tf.train.GradientDescentOptimizer(learning_rate=0.01)
train = optimizer.minimize(cost)  # minimize 하는 구간


# Session 생성
sess = tf.Session()
sess.run(tf.global_variables_initializer()) 
# 변수 선언하고 사용하기전에 초기화 해줘야됨

for step in range(2000):
    sess.run(train)
    if step % 20 == 0:
        print(step, sess.run(cost), sess.run(W),sess.run(b))
        # 잘실행되고있는지 결과를 20번에 한번 출력하도록.
```


결과 !
```python
0 4.477201 [-0.05223601] [0.17080523]
20 0.07695516 [0.69946104] [0.47169712]
40 0.03372201 [0.78045255] [0.47894418]
60 0.030298918 [0.79713756] [0.45923638]
...
1940 3.5577493e-06 [0.99780935] [0.00497998]
1960 3.2310247e-06 [0.9979122] [0.00474595]
1980 2.9347382e-06 [0.99801034] [0.00452292]
```
W와 b가 1과 0으로 수렴함을 알 수 있다.

## Place holder 를 이용한 간단한 Linear Regression 구현
```python
import tensorflow as tf


# x 랑 y 선언
#x_train = [1,2,3]
#y_train = [1,2,3]

W = tf.Variable(tf.random_normal([1]),name='weight')
b = tf.Variable(tf.random_normal([1]),name='bias')
X = tf.placeholder(tf.float32,shape=[None])
Y = tf.placeholder(tf.float32,shape=[None])


# hypothesis 정의. W가 1, b가 0이 나와야함.
hypothesis = X * W + b

# cost 정의
cost = tf.reduce_mean(tf.square(hypothesis - Y))

# minimize
optimizer = tf.train.GradientDescentOptimizer(learning_rate=0.01)
train = optimizer.minimize(cost)  # minimize 하는 구간
## 여기까지 그래프그리미.

# Session 생성
sess = tf.Session()
sess.run(tf.global_variables_initializer()) 
# 변수 선언하고 사용하기전에 초기화 해줘야됨

for step in range(2000):
    cost_val, W_val, b_val, _ = sess.run([cost,W,b,train],
            feed_dict={X: [1,2,3,4,5], Y: [2.1,3.1,4.1,5.1,6.1]})
    if step % 20 == 0:
        print(step, cost_val, W_val, b_val)
        # 잘실행되고있는지 결과를 20번에 한번 출력하도록.


 # 이 TEST코드는 어떻게 TEST해보는지 모르겠다...
print(sess.run(hypothesis, feed_dict={x: {5}})) 
```
대략 W가 1 , b가 1임을 예측 할 수 있다.

결과!
```python
0 0.05110865 [1.1018369] [0.90524894]
20 0.006863064 [1.0538001] [0.90654534]
40 0.0059927986 [1.0500898] [0.9191632]
60 0.005233554 [1.0468086] [0.9310061]
...

1940 1.5554406e-08 [1.0000808] [1.0997084]
1960 1.3591944e-08 [1.0000756] [1.0997275]
1980 1.1926931e-08 [1.0000707] [1.0997449]
```
W와 b가 1에 수렴함을 알 수 있다.
