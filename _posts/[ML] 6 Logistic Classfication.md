[**](/)

𝟷-𝟹-𝟷 {.site-title}
=====

**

**

-   [**☾ 𝙷𝚘𝚖𝚎](/)
-   [**☾ 𝙲𝚊𝚝𝚎𝚐𝚘𝚛𝚒𝚎𝚜](/categories/)
-   [**☾ 𝚃𝚊𝚐𝚜](/tags/)
-   [**☾ 𝙰𝚛𝚌𝚑𝚒𝚟𝚎𝚜](/archives/)
-   **Search

**

**

**

** 0%

[ML] 6 Logistic Classification {.post-title itemprop="name headline"}
==============================

** Posted on 2020-09-03 ** In [Study](/categories/Study/) ,
[ML](/categories/Study/ML/)

[](#Logistic-Classification이란 "Logistic Classification이란")Logistic Classification이란 {#Logistic-Classification이란}
-----------------------------------------------------------------------------------------

둘 중에 하나를 고르는 것. 예를들면 스팸인지 아닌지. 또는 페이스북 피드.
선별해서 재밌고 좋은거만 보여줌. 고로 결과가 0과 1 중 하나로
나타내어져야한다

### [](#Logistic-Hypothesis "Logistic Hypothesis")Logistic Hypothesis {#Logistic-Hypothesis}

logistic hypothesis는 다음과 같이 나타내어진다\
`H(X) = 1/ 1+e^(-W^T*X)`

이 함수는 무조건 0 아니면 1로 수렴하게 된다.

### [](#Cost-function "Cost function")Cost function {#Cost-function}

C( H(x),y ) = ylog(H(x)) - (1-y)log(1-H(x))\
y가 1일 때는 뒤에 있는 항이 없어지고 y가 0일 때는 앞에 있는 항이
없어진다

[](#실습 "실습")실습
--------------------

+--------------------------------------+--------------------------------------+
|     12345678910111213141516171819202 |     import tensorflow as tfx_data =  |
| 122232425262728293031323334353637383 | [[1,2],[2,3],[3,1],[4,3],[5,3],[6,2] |
| 94041424344454647                    | ] # x값이 2개y_data = [[0],[0],[0],[1], |
|                                      | [1],[1]]X = tf.placeholder(tf.float3 |
|                                      | 2, shape=[None, 2])# 들어오는 x값의 갯수Y =  |
|                                      | tf.placeholder(tf.float32, shape=[No |
|                                      | ne, 1])# 나가는 y값의 갯수W = tf.Variable(t |
|                                      | f.random_normal([2,1]), name = 'weig |
|                                      | ht')# x값이 2개, y값이 1개b = tf.Variable( |
|                                      | tf.random_normal([1]),name='bias')#  |
|                                      | logistic hypothesishypothesis = tf.s |
|                                      | igmoid(tf.matmul(X,W)+b)# cost funct |
|                                      | ioncost = -tf.reduce_mean(Y* tf.log( |
|                                      | hypothesis) + (1-Y) * tf.log(1-hypot |
|                                      | hesis))train = tf.train.GradientDesc |
|                                      | entOptimizer(learning_rate=0.01).min |
|                                      | imize(cost)predicted = tf.cast(hypot |
|                                      | hesis > 0.5, dtype=tf.float32)#0.5보다 |
|                                      |  크면 1 (true) 작으면 0 (false)accuracy = |
|                                      |  tf.reduce_mean(tf.cast(tf.equal(pre |
|                                      | dicted, Y),dtype=tf.float32))with tf |
|                                      | .Session() as sess:    sess.run(tf.g |
|                                      | lobal_variables_initializer())    fo |
|                                      | r step in range(10001):        cost_ |
|                                      | val, _ = sess.run([cost, train], fee |
|                                      | d_dict={X: x_data, Y: y_data})       |
|                                      |   if step % 200 == 0:            pri |
|                                      | nt(step, cost_val)    h,c,a = sess.r |
|                                      | un([hypothesis ,predicted,accuracy], |
|                                      | feed_dict={X: x_data, Y: y_data})    |
|                                      |  print("\nHypothesis: ", h,"\nCorrec |
|                                      | t (Y):", c, "\nAccuracy: ",a)# hypot |
|                                      | hesis는 계산한 값. 0.8 , 0.9 ...# predict |
|                                      | ed는 1, 0 으로 나오는 값. 19번째줄# accuracy는  |
|                                      | 예측한 값과 정답 값을 비교해서 맞는지 틀린지 출력 |
+--------------------------------------+--------------------------------------+

[](#csv-파일을-불러와-예측하는-실습 "csv 파일을 불러와 예측하는 실습")csv 파일을 불러와 예측하는 실습
-----------------------------------------------------------------------------------------------------

예를 들어 당뇨에 대해 당뇨를 결정하는 변수가 x1 \~ x3 까지있고, 당뇨병의
유무를 y로 나타낸다고 할때,( 1이면 걸린거 0이면 안걸린거) 다음과 같은
csv 자료가 있다고 가정하고 실습해보자.

`doctor.csv`\
x1 | x2 | x3 | y\
123| 142|412|0\
13| 422|423|0\
83| 2142|532|1

+--------------------------------------+--------------------------------------+
|     12345678910111213141516171819202 |     import tensorflow as tfxy = np.l |
| 122232425262728293031323334353637383 | oadtxt('doctor.csv', delimiter=',',  |
| 940414243444546                      | dtype=np.float32)x_data = xy[:,0:-1] |
|                                      |  # x1,x2,x3 속성만 가져옴y_data = xy[:,[-1 |
|                                      | ]] # y속성만 가져옴X = tf.placeholder(tf.f |
|                                      | loat32, shape=[None, 3])# 들어오는 x값의 갯 |
|                                      | 수Y = tf.placeholder(tf.float32, shap |
|                                      | e=[None, 1])# 나가는 y값의 갯수W = tf.Varia |
|                                      | ble(tf.random_normal([3,1]), name =  |
|                                      | 'weight')# x값이 3개, y값이 1개b = tf.Vari |
|                                      | able(tf.random_normal([1]),name='bia |
|                                      | s')# logistic hypothesishypothesis = |
|                                      |  tf.sigmoid(tf.matmul(X,W)+b)# cost  |
|                                      | functioncost = -tf.reduce_mean(Y* tf |
|                                      | .log(hypothesis) + (1-Y) * tf.log(1- |
|                                      | hypothesis))train = tf.train.Gradien |
|                                      | tDescentOptimizer(learning_rate=0.01 |
|                                      | ).minimize(cost)predicted = tf.cast( |
|                                      | hypothesis > 0.5, dtype=tf.float32)# |
|                                      | 0.5보다 크면 1 (true) 작으면 0 (false)accur |
|                                      | acy = tf.reduce_mean(tf.cast(tf.equa |
|                                      | l(predicted, Y),dtype=tf.float32))wi |
|                                      | th tf.Session() as sess:    sess.run |
|                                      | (tf.global_variables_initializer())  |
|                                      |    for step in range(10001):         |
|                                      | cost_val, _ = sess.run([cost, train] |
|                                      | , feed_dict={X: x_data, Y: y_data})  |
|                                      |        if step % 200 == 0:           |
|                                      |   print(step, cost_val)    h,c,a = s |
|                                      | ess.run([hypothesis ,predicted,accur |
|                                      | acy],feed_dict={X: x_data, Y: y_data |
|                                      | })    print("\nHypothesis: ", h,"\nC |
|                                      | orrect (Y):", c, "\nAccuracy: ",a)#  |
|                                      | hypothesis는 계산한 값. 0.8 , 0.9 ...# pr |
|                                      | edicted는 1, 0 으로 나오는 값. 19번째줄# accur |
|                                      | acy는 예측한 값과 정답 값을 비교해서 맞는지 틀린지 출력 |
+--------------------------------------+--------------------------------------+

[\# study](/tags/study/) [\# ML](/tags/ML/) [\# python](/tags/python/)

[** [pwnable.kr] bof
풀이](/2020/09/02/%5Bpwnable.kr%5D%20bof%20%ED%92%80%EC%9D%B4/ "[pwnable.kr] bof 풀이")

[[ML] 7 Softmax Regression
**](/2020/09/03/%5BML%5D%207%20Softmax%20Regression/ "[ML] 7 Softmax Regression")

-   Table of Contents
-   Overview

1.  [1. Logistic
    Classification이란](#Logistic-Classification%EC%9D%B4%EB%9E%80)
    1.  [1.1. Logistic Hypothesis](#Logistic-Hypothesis)
    2.  [1.2. Cost function](#Cost-function)

2.  [2. 실습](#%EC%8B%A4%EC%8A%B5)
3.  [3. csv 파일을 불러와 예측하는
    실습](#csv-%ED%8C%8C%EC%9D%BC%EC%9D%84-%EB%B6%88%EB%9F%AC%EC%99%80-%EC%98%88%EC%B8%A1%ED%95%98%EB%8A%94-%EC%8B%A4%EC%8A%B5)

![𝟷-𝟹-𝟷](/images/avatar.gif)

𝟷-𝟹-𝟷

[15 𝙿𝚘𝚜𝚝𝚜](/archives)

[4 𝙲𝚊𝚝𝚎𝚐𝚘𝚛𝚒𝚎𝚜](/categories/)

[7 𝚃𝚊𝚐𝚜](/tags/)

[**𝚒𝚛𝚘𝚗𝚛𝚊𝚒𝚗𝟹𝟺@𝚐𝚖𝚊𝚒𝚕.𝚌𝚘𝚖](/ironrain34@gmail.com "𝚒𝚛𝚘𝚗𝚛𝚊𝚒𝚗𝟹𝟺@𝚐𝚖𝚊𝚒𝚕.𝚌𝚘𝚖 → ironrain34@gmail.com")

© 2020 ** 𝟷-𝟹-𝟷

Powered by [Hexo](https://hexo.io/) &
[NexT.Muse](https://muse.theme-next.org/)
