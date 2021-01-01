# tf中的padding方式SAME和VALID有什么区别?


Tensorflow中的padding有两种方式，其中的SAME方式比较特殊，可能产生非对称pad的情况，之前在验证一个tensorflow的网络时踩到这个坑。

<!--more-->

## Tensorflow的计算公式

### 二维卷积接口

  ```python
  tf.nn.conv2d(
      input, filters, strides, padding, data_format='NHWC', dilations=None, name=None
  )
  ```

### padding计算公式

需要注意padding的配置，如果是字符串就有`SAME`和`VALID`两种配置，如果是数字list就明确表明padding在各个维度的数量。

首先，`padding`如果表示确切的数字，其维度是input维度的2倍，因为每个维度两个边需要补pad，比如宽度的左边和右边，高度的上边和下边，但是tensorflow中不会给N维度以及C维度补pad，仅仅在W和H维度补pad，因此对于`NHWC`，`padding =[[0, 0], [pad_top, pad_bottom], [pad_left, pad_right], [0, 0]] `，对于`NCHW`的pad的顺序换过来。

  然后，如果输入的是字符串选项，补的pad都可以映射到padding这个参数上，

  - `VALID`模式表示不在任何维度补pad，等价于`padding =[[0, 0], [0, 0], [0, 0], [0, 0]]`；
  
  - `SAME`模式表示在`stride`的尺度下，`Wo`与`Wi`保持在stride$S$下保持一致（以宽度维度为例），需要满足如下关系
    $$
    W_{o}=\left\lceil\frac{W_{i}}{S}\right\rceil
    $$
    我们知道如果`dilation = 1`，那么在某个维度上，卷积的输出宽度$W_i$、输出宽度$W_o$和步长$S$，在没有pad的情况下，满足如下的关系式
    $$
    W o=\left\lfloor\frac{W i-W_{k}}{S}\right\rfloor+1
    $$
    我们以补最小程度的$P_a$为基准，于是有关系式
    $$
    W o=\frac{W i+P_{a}-W_{k}}{S}+1
    $$
    反推过来得到
    $$
    P_{a}=\left(W_{o}-1\right) S+W_{k}-W_{i}
    $$
    这是需要补的总的pad，tensorflow的补充pad是尽量两边对称的，
  
    - 如果$P_a$是偶数，那么两边都补$P_l = P_a/2$；
    - 如果$P_a$是奇数，那么左边补$P_l = \lfloor{P_a/2}\rfloor$，右边是$P_r = P_a-P_l$。
    
    

  参考如下的代码

~~~python
inputH, inputW = 7, 8
strideH, strideW = 3, 3
filterH = 4
filterW = 4
inputC = 16
outputC = 3
inputData = np.ones([1,inputH,inputW,inputC]) # format [N, H, W, C]
filterData = np.float16(np.ones([filterH, filterW, inputC, outputC]) - 0.33)
strides = (1, strideH, strideW, 1)
convOutputSame = tf.nn.conv2d(inputData, filterData, strides, padding='SAME')
convOutput = tf.nn.conv2d(inputData, filterData, strides, padding=[[0,0],[1,2],[1,1],[0,0]]) # padded input data
print("output1, ", convOutputSame)
print("output2, ", convOutput)
print("Sum of a - b is ", np.sum(np.square(convOutputSame - convOutput)))
~~~

计算结果是

~~~python
    output1,  tf.Tensor(
    [[[[ 96.46875  96.46875  96.46875]
       [128.625   128.625   128.625  ]
       [ 96.46875  96.46875  96.46875]]

      [[128.625   128.625   128.625  ]
       [171.5     171.5     171.5    ]
       [128.625   128.625   128.625  ]]

      [[ 64.3125   64.3125   64.3125 ]
       [ 85.75     85.75     85.75   ]￼
       [ 64.3125   64.3125   64.3125 ]]]])

    output2,  tf.Tensor(
    [[[[ 96.46875  96.46875  96.46875]
       [128.625   128.625   128.625  ]
       [ 96.46875  96.46875  96.46875]]

      [[128.625   128.625   128.625  ]
       [171.5     171.5     171.5    ]
       [128.625   128.625   128.625  ]]

      [[ 64.3125   64.3125   64.3125 ]
       [ 85.75     85.75     85.75   ]
       [ 64.3125   64.3125   64.3125 ]]]], shape=(1, 3, 3, 3), dtype=float64)
    Sum of a - b is  0.0
~~~

## ONNX计算公式

onnx的接口，参考IR定义如下

  ```c++
std::function<void(OpSchema&)> ConvOpSchemaGenerator(const char* filter_desc) {
    return [=](OpSchema& schema) {
      std::string doc = R"DOC(
  The convolution operator consumes an input tensor and {filter_desc}, and
  computes the output.)DOC";
      ReplaceAll(doc, "{filter_desc}", filter_desc);
      schema.SetDoc(doc);
      schema.Input(
          0,
          "X",
          "Input data tensor from previous layer; "
          "has size (N x C x H x W), where N is the batch size, "
          "C is the number of channels, and H and W are the "
          "height and width. Note that this is for the 2D image. "
          "Otherwise the size is (N x C x D1 x D2 ... x Dn). "
          "Optionally, if dimension denotation is "
          "in effect, the operation expects input data tensor "
          "to arrive with the dimension denotation of [DATA_BATCH, "
          "DATA_CHANNEL, DATA_FEATURE, DATA_FEATURE ...].",
          "T");
      schema.Input(
          1,
          "W",
          "The weight tensor that will be used in the "
          "convolutions; has size (M x C/group x kH x kW), where C "
          "is the number of channels, and kH and kW are the "
          "height and width of the kernel, and M is the number "
          "of feature maps. For more than 2 dimensions, the "
          "kernel shape will be (M x C/group x k1 x k2 x ... x kn), "
          "where (k1 x k2 x ... kn) is the dimension of the kernel. "
          "Optionally, if dimension denotation is in effect, "
          "the operation expects the weight tensor to arrive "
          "with the dimension denotation of [FILTER_OUT_CHANNEL, "
          "FILTER_IN_CHANNEL, FILTER_SPATIAL, FILTER_SPATIAL ...]. "
          "X.shape[1] == (W.shape[1] * group) == C "
          "(assuming zero based indices for the shape array). "
          "Or in other words FILTER_IN_CHANNEL should be equal to DATA_CHANNEL. ",
          "T");
      schema.Input(
          2,
          "B",
          "Optional 1D bias to be added to the convolution, has size of M.",
          "T",
          OpSchema::Optional);
      schema.Output(
          0,
          "Y",
          "Output data tensor that contains the result of the "
          "convolution. The output dimensions are functions "
          "of the kernel size, stride size, and pad lengths.",
          "T");
      schema.TypeConstraint(
          "T",
          {"tensor(float16)", "tensor(float)", "tensor(double)"},
          "Constrain input and output types to float tensors.");
      schema.Attr(
          "kernel_shape",
          "The shape of the convolution kernel. If not present, should be inferred from input W.",
          AttributeProto::INTS,
          OPTIONAL);
      schema.Attr(
          "dilations",
          "dilation value along each spatial axis of the filter. If not present, the dilation defaults is 1 along each spatial axis.",
          AttributeProto::INTS,
          OPTIONAL);
      schema.Attr(
          "strides",
          "Stride along each spatial axis. If not present, the stride defaults is 1 along each spatial axis.",
          AttributeProto::INTS,
          OPTIONAL);
      schema.Attr(
          "auto_pad",
          auto_pad_doc,
          AttributeProto::STRING,
          std::string("NOTSET"));
      schema.Attr(
          "pads",
          pads_doc,
          AttributeProto::INTS,
          OPTIONAL);
      schema.Attr(
          "group",
          "number of groups input channels and output channels are divided into.",
          AttributeProto::INT,
          static_cast<int64_t>(1));
      schema.TypeAndShapeInferenceFunction([](InferenceContext& ctx) {
        propagateElemTypeFromInputToOutput(ctx, 0, 0);
        convPoolShapeInference(ctx, true, false, 0, 1);
      });
    };
  }
  ```

  需要注意上面的`auto_pad`选项，与tensorflow类似有3个选项

  - `NOTSET`，同tensorflow的`VALID`

  - `SAME_UPPER `或者`SAME_LOWER`，这里的内容可以参考onnx文件`defs.cc`中的代码

    ```c++
    std::vector<int64_t> pads;
      if (getRepeatedAttribute(ctx, "pads", pads)) {
        if (pads.size() != n_input_dims * 2) {
          fail_shape_inference("Attribute pads has incorrect size");
        }
      } else {
        pads.assign(n_input_dims * 2, 0); // pads的size是输入维度的2倍
        const auto* auto_pad_attr = ctx.getAttribute("auto_pad");
        if ((nullptr != auto_pad_attr) && (auto_pad_attr->s() != "VALID")) { // 如果pad mode是SAME_UPPER或者SAME_LOWER则进入该分支
          int input_dims_size = static_cast<int>(n_input_dims);
          for (int i = 0; i < input_dims_size; ++i) { // 计算每个axis的pads
            int64_t residual = 0;
            int64_t stride = strides[i];
            if (stride > 1) { // 如果stride == 1，那么total_pad就是Wk - Stride = Wk - 1
              if (!input_shape.dim(2 + i).has_dim_value()) {
                continue;
              }
              residual = input_shape.dim(2 + i).dim_value();
              while (residual >= stride) {
                residual -= stride;
              }
            }
            int64_t total_pad = residual == 0 ? effective_kernel_shape[i] - stride : effective_kernel_shape[i] - residual; // effective_kernel_shape = Wk
            if (total_pad < 0)
              total_pad = 0;
            int64_t half_pad_small = total_pad >> 1;
            int64_t half_pad_big = total_pad - half_pad_small;
            if (auto_pad_attr->s() == "SAME_UPPER") {           // pad mode is here
              pads[i] = half_pad_small;
              pads[i + input_dims_size] = half_pad_big;
            } else if (auto_pad_attr->s() == "SAME_LOWER") {
              pads[i] = half_pad_big;
              pads[i + input_dims_size] = half_pad_small;
            }
          }
        }
      }
    ```

    上面的代码里面最难理解14~23行，其实这里计算total_pad就是tensorflow中的$P_a$，以上面的公式，做更进一步的推导，
    $$
    \begin{equation}
    \begin{aligned}
    P_{a}&=\left(W_{o}-1\right) S+W_{k}-W_{i}\\
    &=(\left\lceil\frac{W_{i}}{S}\right\rceil - 1)S+W_{k}-W_{i}\\
    \end{aligned}
    \end{equation}
    $$
    
    下面的分析有两种情况，对应代码第23行，
    
    - 如果$W_i$是$S$的整数倍，那么$W_i = nS$，带入上面的公式有$P_a = W_k - S$；
    - 如果$W_i$不是$S$的整数倍，那么$W_i = nS+m, m \gt 0$，带入上面的公式有$P_a = W_k - m$，这个$m$就是$W_i$被Stride相除之后的余数，即代码中的`residual`。
    
    `SAME_UPPER`和`SAME_LOWER`对应$P_a$是奇数的情况，如果是偶数，结果一样，如果是奇数，那么`SAME_UPPER`放小半部分$\lfloor{P_a/2}\rfloor$，`SAME_LOWER`放大半部分$P_a - \lfloor{P_a/2}\rfloor$。

## 举例

在[python - What is the difference between 'SAME' and 'VALID' padding in tf.nn.max\_pool of tensorflow? - Stack Overflow](https://stackoverflow.com/questions/37674306/what-is-the-difference-between-same-and-valid-padding-in-tf-nn-max-pool-of-t)上有一个比较具体的例子，可以看到，使用`SAME`模式可以使得stride刚好完整取完所有的input而不会有剩余或者短缺。

  - `VALID` 模式

    ```shell
    inputs:         1  2  3  4  5  6  7  8  9  10 11 (12 13)
                    |________________|                dropped
                                   |_________________|
    ```

  - `SAME`模式

    ```shell
                   pad|                                      |pad
       inputs:      0 |1  2  3  4  5  6  7  8  9  10 11 12 13|0  0
                   |________________|
                                  |_________________|
                                                 |________________|
    
    ```
    在这个例子中$W_i = 13, W_k = 6, S = 5$。

## 参考资料

- [TensorFlow中CNN的两种padding方式“SAME”和“VALID” - wuzqChom的博客 - CSDN博客](https://blog.csdn.net/wuzqChom/article/details/74785643)
- [python - What is the difference between 'SAME' and 'VALID' padding in tf.nn.max\_pool of tensorflow? - Stack Overflow](https://stackoverflow.com/questions/37674306/what-is-the-difference-between-same-and-valid-padding-in-tf-nn-max-pool-of-t)
- [What does the 'same' padding parameter in convolution mean in TensorFlow? - Quora](https://www.quora.com/What-does-the-same-padding-parameter-in-convolution-mean-in-TensorFlow)


~~~

~~~
