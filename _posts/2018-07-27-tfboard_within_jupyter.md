---
title: 텐서보드를 주피터로 띄우기 - tensorboard within Jupyter
date: 2018-07-27
tags: deeplearning
category: programming
---

데이터 분석을 공부할 때는 주로 Jupyter notebook을 이용하게 된다. 최근에 다시 tensorflow를 복습하게 되어서 쓰다 보니, 간단하게 주피터 내에서 텐서보드를 띄우고 싶었다. 예전에 찾았던 것 같은데, 앞으로 또 찾을 일을 대비해 여기에 기록한다.


아래 두 함수만 있으면 된다.

```python
from IPython.display import clear_output, Image, display, HTML

def strip_consts(graph_def, max_const_size=32):
    """Strip large constant values from graph_def."""
    strip_def = tf.GraphDef()
    for n0 in graph_def.node:
        n = strip_def.node.add()
        n.MergeFrom(n0)
        if n.op == 'Const':
            tensor = n.attr['value'].tensor
            size = len(tensor.tensor_content)
            if size > max_const_size:
                tensor.tensor_content = "<stripped %d bytes>"%size
    return strip_def

def show_graph(graph_def, max_const_size=32):
    """Visualize TensorFlow graph."""
    if hasattr(graph_def, 'as_graph_def'):
        graph_def = graph_def.as_graph_def()
    strip_def = strip_consts(graph_def, max_const_size=max_const_size)
    code = """
        <script src="//cdnjs.cloudflare.com/ajax/libs/polymer/0.3.3/platform.js"></script>
        <script>
          function load() {{
            document.getElementById("{id}").pbtxt = {data};
          }}
        </script>
        <link rel="import" href="https://tensorboard.appspot.com/tf-graph-basic.build.html" onload=load()>
        <div style="height:600px">
          <tf-graph-basic id="{id}"></tf-graph-basic>
        </div>
    """.format(data=repr(str(strip_def)), id='graph'+str(np.random.rand()))

    iframe = """
        <iframe seamless style="width:1200px;height:620px;border:0" srcdoc="{}"></iframe>
    """.format(code.replace('"', '&quot;'))
    display(HTML(iframe))
```

실행을 위한 예시 코드


```python
def relu(X):
    w_shape = (int(X.get_shape()[1]), 1)
    w = tf.Variable(tf.random_normal(w_shape), name="weights")
    b = tf.Variable(0.0, name="bias")
    z = tf.add(tf.matmul(X, w), b, name="z")
    return tf.maximum(z, 0., name="relu")

n_features = 3
X = tf.placeholder(tf.float32, shape=(None, n_features), name="X")

relus = [relu(X) for i in range(2)]
output = tf.add_n(relus, name="output")
```

실행


```python
show_graph(tf.get_default_graph().as_graph_def())
```

![img](/assets/img/tensorboard_on_jupyter.png)

interaction까지 동작하는 tensorboard가 Jupyter notebook 환경에 띄워진다. thank you stackoverflow!
