---
title: 오늘의 에러 - 180813
date: 2018-08-13
tags: Error deeplearning
category: programming
---

신경망 모델을 클래스로 구현해보던 중 `Saver`에서 에러(`ValueError: No variables to save`)가 발생했다.


## 코드 일부

```python
class model():
    def __init__(self, NN, data):
        self.NN = NN
        self.data = data
        self.sess = tf.Session()
        self.saver = tf.train.Saver()

        self.X = tf.placeholder(tf.float32, shape=(None, 28 * 28), name="X")
        self.y = tf.placeholder(tf.int64, shape=(None), name="y")

        self.y_hat = self.NN(self.X)

        self.loss = tf.reduce_mean(tf.nn.sparse_softmax_cross_entropy_with_logits(
            labels=self.y, logits=self.y_hat), name="loss")
        self.optimizer = tf.train.GradientDescentOptimizer(
            learning_rate=0.01).minimize(self.loss)
        self.accuracy = tf.reduce_mean(
            tf.cast(tf.nn.in_top_k(self.y_hat, self.y, 1), tf.float32))
```


## 에러 메세지

```console
Traceback (most recent call last):
  File "tf_NN_class.py", line 124, in <module>
    mymodel = model(NN(), mnist)
  File "tf_NN_class.py", line 36, in __init__
    self.saver = tf.train.Saver()
  File "/home/mk/anaconda3/lib/python3.6/site-packages/tensorflow/python/training/saver.py", line 1284, in __init__
    self.build()
  File "/home/mk/anaconda3/lib/python3.6/site-packages/tensorflow/python/training/saver.py", line 1296, in build
    self._build(self._filename, build_save=True, build_restore=True)
  File "/home/mk/anaconda3/lib/python3.6/site-packages/tensorflow/python/training/saver.py", line 1321, in _build
    raise ValueError("No variables to save")
ValueError: No variables to save

```

`Saver`는 텐서플로우의 그래프를 저장하는 변수인데, 저장할 변수가 없다고 뜬다. 이유는 실제로 위 코드에서는 그래프가 만들어지기 전에 저장을 해버렸기 때문. 그래프만 만들어두고 실제 연산은 나중에 되는 텐서플로우에 고새 익숙해져서 (+ 코드를 깔끔하게 하려다가) 이런 에러가 발생했다. 덕분에 `Saver`호출과 동시에 그때까지 만들어진 그래프를 저장한다는 것을 깨달았다.<br>





## 고친 코드

```python
class model():
    def __init__(self, NN, data):
        self.NN = NN
        self.data = data

        self.X = tf.placeholder(tf.float32, shape=(None, 28 * 28), name="X")
        self.y = tf.placeholder(tf.int64, shape=(None), name="y")

        self.y_hat = self.NN(self.X)

        self.loss = tf.reduce_mean(tf.nn.sparse_softmax_cross_entropy_with_logits(
            labels=self.y, logits=self.y_hat), name="loss")
        self.optimizer = tf.train.GradientDescentOptimizer(
            learning_rate=0.01).minimize(self.loss)
        self.accuracy = tf.reduce_mean(
            tf.cast(tf.nn.in_top_k(self.y_hat, self.y, 1), tf.float32))

        self.sess = tf.Session()
        self.saver = tf.train.Saver()
```
