# Pooling Layer

Recall that, in the image object edge detection application covered in the ["Two-dimensional Convolutional Layer" ](conv-layer.md) section, we constructed a convolution kernel and accurately located the position of the pixel change. Set the element `i` line `j` column of any two-dimensional array `X` as `X[i, j]`. If the output of the convolution kernel we constructed is `Y[i, j]=1`, then the values of `X[i, j]` and `X[i, j+1]` in the input are different. This may mean that the edge of the object passes between these two elements. However, in the actual image, the objects we are interested in will not always appear in a fixed position: even if we continuously shoot the same object, there is a high possibility of pixel shift. This causes the output corresponding to the same edge to appear in different locations in the convolution output `Y`, resulting in problems for later pattern recognition.

This section introduces pooling layers, which were proposed to alleviate the excessive sensitivity of the convolutional layer to location.

## Two-dimensional Maximum Pooling Layers and Average Pooling Layers

Like the convolutional layer, the pooling layer computes the output for each element in a fixed-shape window (also known as a pooling window) of input data. Different from the cross-correlation computation of the inputs and kernels in the convolutional layer, the pooling layer directly calculates the maximum or average value of the elements in the pooling window. This operation is also called maximum pooling or average pooling. In maximum pooling, the pooling window starts from the top left of the input array, and slides in the input array from left to right and top to bottom. When the pooling window slides to a certain position, the maximum value of the input subarray in the window is the element at the corresponding location in the output array.

![Maximum pooling with a pooling window shape of $2\times 2$. The shaded portions represent the first output element and the input element used for its computation: $\max(0,1,3,4)=4$](../img/pooling.svg)

The output array in Figure 5.6 has a height of 2 and a width of 2. The four elements are derived from the maximum value of $\text{max}$:

$$
\max(0,1,3,4)=4,\\
\max(1,2,4,5)=5,\\
\max(3,4,6,7)=7,\\
\max(4,5,7,8)=8.\\
$$


Average pooling works like maximum pooling, only with the maximum operator replaced by the average operator. The pooling layer with a pooling window shape of $p \times q$ is called the $p \times q$ pooling layer. The pooling operation is called $p \times q$ pooling.

Let us return to the object edge detection example mentioned at the beginning of this section. Now we will use the output of the convolutional layer as the input for $2\times 2$ maximum pooling. Set the convolutional layer input as `X` and the pooling layer output as `Y`. Whether or not the values of `X[i, j]` and `X[i, j+1]` are different, or `X[i, j+1]` and `X[i, j+2]` are different, the pooling layer outputs all include `Y[i, j]=1`. That is to say, using the $2\times 2$ maximum pooling layer, we can still detect if the pattern recognized by the convolutional layer moves no more than one element in height and width.

As shown below, we implement the forward computation of the pooling layer in the `pool2d` function. This function is very similar to the `corr2d` function in the ["Two-dimensional Convolutional Layer"](conv-layer.md) section. The only difference lies in the computation of the output `Y`.

```{.python .input  n=11}
from mxnet import nd
from mxnet.gluon import nn

def pool2d(X, pool_size, mode='max'):
    p_h, p_w = pool_size
    Y = nd.zeros((X.shape[0] - p_h + 1, X.shape[1] - p_w + 1))
    for i in range(Y.shape[0]):
        for j in range(Y.shape[1]):
            if mode == 'max':
                Y[i, j] = X[i: i + p_h, j: j + p_w].max()
            elif mode == 'avg':
                Y[i, j] = X[i: i + p_h, j: j + p_w].mean()       
    return Y
```

We can construct the input array `X` in Figure 5.6 to validate the output of the two-dimensional maximum pooling layer.

```{.python .input  n=13}
X = nd.array([[0, 1, 2], [3, 4, 5], [6, 7, 8]])
pool2d(X, (2, 2))
```

At the same time, we experiment with the average pooling layer.

```{.python .input  n=14}
pool2d(X, (2, 2), 'avg')
```

## Padding and Stride

Like the convolutional layer, the pooling layer can also change the output shape by padding the two sides of the input height and width and adjusting the window stride. The pooling layer works in the same way as the convolutional layer in terms of padding and strides. We will demonstrate the use of padding and stride in the pooling layer through the two-dimensional maximum pooling layer MaxPool2D in the `nn` module. We first construct an input data of shape `(1, 1, 4, 4)`, where the first two dimensions are batch and channel.

```{.python .input  n=15}
X = nd.arange(16).reshape((1, 1, 4, 4))
X
```

By default, the stride in the `MaxPool2D` class has the same shape as the pooling window. Below, we use a pooling window of shape `(3, 3)`, so we get a stride shape of `(3, 3)` by default.

```{.python .input  n=16}
pool2d = nn.MaxPool2D(3)
pool2d(X)  # Because there are no model parameters in the pooling layer, we do not need to call the parameter initialization function.
```

The stride and padding can be manually specified.

```{.python .input  n=7}
pool2d = nn.MaxPool2D(3, padding=1, strides=2)
pool2d(X)
```

Of course, we can also specify a non-square pooling window and designate the padding and stride on the height and width, respectively.

```{.python .input  n=8}
pool2d = nn.MaxPool2D((2, 3), padding=(1, 2), strides=(2, 3))
pool2d(X)
```

## Multiple Channels

When processing multi-channel input data, the pooling layer pools each input channel separately, rather than adding the inputs of each channel by channel as in a convolutional layer. This means that the number of output channels for the pooling layer is the same as the number of input channels. Below, we will concatenate arrays `X` and `X+1` on the channel dimension to construct an input with 2 channels.

```{.python .input  n=9}
X = nd.concat(X, X + 1, dim=1)
X
```

As we can see, the number of output channels is still 2 after pooling.

```{.python .input  n=10}
pool2d = nn.MaxPool2D(3, padding=1, strides=2)
pool2d(X)
```

## Summary

* Taking the input elements in the pooling window, the maximum pooling operation assigns the maximum value as the output and the average pooling operation assigns the average value as the output.
* One of the major functions of a pooling layer is to alleviate the excessive sensitivity of the convolutional layer to location.
* We can specify the padding and stride for the pooling layer.
* The pooling layer's number of output channels is the same as the number of input channels.


## exercise

* Analyze the computational complexity of the pooling layer. If we assume that a pooling layer has an input shape of $c\times h\times w$, and the pooling window has a shape of $p_h\times p_w$, with a padding of $(p_h, p_w)$ and a stride of $(s_h, s_w)$,  what is the complexity of forward computation for this pooling layer?
* What differences might there be between the maximum pooling layer and average pooling layer?
* Do you think the idea of a minimum pooling layer makes sense?

## Scan the QR Code to Access [Discussions](https://discuss.gluon.ai/t/topic/6406)

![](../img/qr_pooling.svg)