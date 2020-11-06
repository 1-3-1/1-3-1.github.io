---
layout: post
title: "[ML] 2 Tensorflow "
date: 2020-11-02
categories: ETC
tags: [MahineLearning,Development]
---
[ML] 2 Tensorflow

## Tensorflow 설치
1. python 3.5 버전을 설치한다 ( python 3.8 버전은 tf 1.x.x를 지원하지않는다고함)
2. tensorflow 설치 ( 아래 중 하나를 설치하면되는데 나는 둘다 안돼서 둘다 했음)
하는 도중 에러가 발생하면 하단에 작성해둔 에러 해결법을 참고바란다


`pip install --upgrade tensorflow`
`pip install --upgrade tensorflow-gpu`


3. tf가 잘 설치되었는지 확인 ( python 실행)
```python
import tensorflow as 
tf
```

결과!
```python
tf.__version__
 '1.0.0'
```


## Hello tensorflow!
```python
import tensorflow as tf

hello = tf.constant("hello")
sess = tf.Session()
print(sess.run(hello))
```

결과 !

```python
b'hello'
```


## Graph
1. 그래프를 빌드해줌
2. 세션을 생성
3. 출력

```python
import tensorflow as tf

node1 = tf.constant(3.0,tf.float32)
node2 = tf.constant(4.0)
node3 = tf.add(node1,node2)

sess = tf.Session() # 항상  session을 만들고 출력해줘야함.
print(sess.run([node1,node2]))
print(sess.run([node3]))
```

결과 !

```python
[3.0, 4.0]
[7.0]
```

## Placeholder
나중에 값을 지정해줌. placeholder라는 노드로 지정해주는 것.

```python
import tensorflow as tf

sess = tf.Session()
a = tf.placeholder(tf.float32)
b = tf.placeholder(tf.float32)
adder_node = a+b

print(sess.run(adder_node, feed_dict={a: 3, b: 4.5}))
print(sess.run(adder_node, feed_dict={a: [1,3], b: [2,4]}))
```

결과 !
```python
7.5
[3. 7.]
```



## ERROR 해결
- Tensorflow Import 에러

```python
import tensorflow as tf
2020-08-24 02:42:06.151240: W tensorflow/stream_executor/platform/default/dso_loader.cc:59] Could not load dynamic library 'cudart64_101.dll'; dlerror: cudart64_101.dll not found
2020-08-24 02:42:06.164988: I tensorflow/stream_executor/cuda/cudart_stub.cc:29] Ignore above cudart dlerror if you do not have a GPU set up on your machine.
```


다음과 같이 설치하면 에러 없어짐
`pip install --upgrade tensorflow-cpu`


그 외 에러 해결 시 도움을 받은 페이지들
[tensorflow import error](https://nogadaworks.tistory.com/129)
[tensorflow import error - numpy 관련](https://antilibrary.org/2259)