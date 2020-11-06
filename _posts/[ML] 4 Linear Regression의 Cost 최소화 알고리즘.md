---
layout: post
title: "[ML] 4 Linear Regression의 cost 최소화 알고리즘"
date: 2020-11-02
categories: ETC
tags: [MahineLearning,Development]
---

## Cost Function (비용함수)
cost는 예측하는 값과 실제 결과 값의 차이를 나타내는 값이며 cost function은 차이를 나타내는 함수이다. 따라서 cost값이 최소화인 W값을 찾는 것이 머신러닝의 목표라 할 수 있다. (정확한 w값을 가질 수록 0에 수렴)

### Cost Function 구현

```python
import tensorflow as tf
import matplotlib.pyplot as plt

X = [1,2,3]
Y = [1,2,3]

W = tf.placeholder(tf.float32)

# H(x) = Wx . b는 생략했음.
hypothesis = X*W

# cost(W) 함수
cost = tf.reduce_mean(tf.square(hypothesis - Y))


sess = tf.Session()
sess.run(tf.global_variables_initializer())

#그래프를 저장할 리스트만들기 
W_val = []
cost_val = []

# -30 에서 50 까지 움직이면서 cost와 W가 어떻게 움직이는지 확인.
# 실질적으로 곱하기 0.1을 해서 -3 ~ 5 까지임.
for i in range(-30,50):
    feed_W = i*0.1
    curr_cost, curr_W = sess.run([cost,W],feed_dict = {W: feed_W})
    W_val.append(curr_W)
    cost_val.append(curr_cost)

# 그래프 그리기
plt.plot(W_val,cost_val)
plt.show()
```


## Gradient descent alorithm(경사하강법)
Cost Function의 최솟값을 구하기 위해 cost(W)를 미분하여 기울기가 0인 W를 점차적으로 찾아내는 알고리즘.

### 경사하강법 구현 - 기본
```python
import tensorflow as tf

x_data = [1,2,3]
y_data = [1,2,3]

W = tf.Variable(tf.random_normal([1]),name='weight')
X = tf.placeholder(tf.float32)
Y = tf.placeholder(tf.float32)

# H(x) = Wx . b는 생략했음.
hypothesis = X*W

# cost(W) 함수
cost = tf.reduce_mean(tf.square(hypothesis - Y))


# Minimize 
learning_rate = 0.1
gradient = tf.reduce_mean((W*X -Y)*X)
descent = W - learning_rate * gradient
update = W.assign(descent)

sess = tf.Session()
sess.run(tf.global_variables_initializer())


for step in range(21):
   sess.run(update,feed_dict={X: x_data, Y: y_data})
   print(step, sess.run(cost, feed_dict={X: x_data, Y: y_data}),sess.run(W))
```

결과 !
```python
1 0.054293144 [0.8921378]
2 0.015443361 [0.94247353]
3 0.0043927873 [0.9693192]
4 0.0012495006 [0.9836369]
...
18 2.984753e-11 [0.9999975]
19 7.716494e-12 [0.9999987]
20 2.3874236e-12 [0.9999993]
```

step / cost / W값 순으로 나열되어있다.

갈수록 cost값이 작아지고 W는 1에 수렴함을 알 수 있다

### 경사하강법 구현 - optimizer
```python
import tensorflow as tf

X = [1,2,3]
Y = [1,2,3]

W = tf.Variable(5.0)

# H(x) = Wx . b는 생략했음.
hypothesis = X*W

# cost(W) 함수
cost = tf.reduce_mean(tf.square(hypothesis - Y))

# Minimize 
optimizer = tf.train.GradientDescentOptimizer(learning_rate=0.1)
train = optimizer.minimize(cost)

sess = tf.Session()
sess.run(tf.global_variables_initializer())

for step in range(100):
   print(step, sess.run(W))
   sess.run(train)
```

결과 !
```python
0 5.0
1 1.2666664
2 1.0177778
3 1.0011852
...
97 1.0
98 1.0
99 1.0
```
굉장히 빠르게 학습하는편.