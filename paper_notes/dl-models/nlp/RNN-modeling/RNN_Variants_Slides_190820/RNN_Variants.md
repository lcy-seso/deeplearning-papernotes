<!-- $theme: gaia -->
<!-- *template: invert -->

# ==RNN Variants==

###### Cao Ying

---

<!-- *template: invert -->

<font size=5>

# ==Outlines==

1. _Motivations and background_
    - Challenges of RNNs from the ML perspective
    - Challenges of RNNs from the system perspective
1. _RNN variants_
    - [Dilated LSTM](https://arxiv.org/abs/1710.02224) (NIPS 2017)
    - [ClockworkRNN](https://arxiv.org/abs/1402.3511) (2014)
    - [Multi-dimensional RNN](https://arxiv.org/pdf/0705.2011.pdf) (2007)
    - [Grid LSTM](https://arxiv.org/abs/1507.01526) (2015)
    - [Hierarchical Multiscale Recurrent Neural Networks](https://openreview.net/forum?id=S1di0sfgl) (ICLR 2017)
    - [ON-LSTM](https://openreview.net/pdf?id=B1l6qiR5F7) (ICLR 2019's best paper award)
1. _Summary & Disscussion_

</font>

---

# Motivations

<font size=5>

1. RNNs are notorious for their sequential execution natures. Nevertheless, is there no parallelism at all in recurrent models?
1. RNN is a wide spectrum of models more than LSTM/GRU. Is there common semantics to describe general-purpose RNN computation, and the description is able to be optimized?

</font>

---

<font size=5>

## Background: Challenges of sequence modeling

1. The vanishing gradient problems is originated from BP algorithm.
    - Existing solutions
        1. Truncated BPTT
        1. gate
        1. skip connections
        1. normalization
1. How to design RNNs to adaptively learn patterns span over multiple and variable time step?
1. How to design RNNs to adaptively learn underlying hierarchically structured features?

</font>

---

<font size=5>

# Recap RNN computation

<p align="center">
<img src="images/Recurrent_neural_network_unfold.svg.png">
</p>

<font size=4>

1. All RNNs are made up of three parts:
    1. _input-to-hidden_ projection $U$
    1. <span style="background-color:#ACD6FF;"> _hidden-to-hidden_ projection $V$</span>
    1. _hidden-to-output_ projection $W$
1. <span style="background-color:#ACD6FF;">hidden-to-hidden projection $U$</span> is intrinsically sequential.
1. the computation of RNN is dominated by MMs, and GEMM works poorly for small MMs.

</font>
</font>

---

<!-- *template: invert -->

<font size=5>

# ==Related work for RNN optimizations==
1. [Investigating performance of GPU BLAS Libraries](http://svail.github.io/rnn_perf/)
1. [Persistent RNNs: 30 times faster RNN layers at small mini-batch sizes](http://svail.github.io/persistent_rnns/)
1. [DeepCpu](https://www.usenix.org/conference/atc18/presentation/zhang-minjia)

</font>

---

<font size=5>

# Gradient vanishing/explosion problem and LSTM

RNNs form a long chain over which gradient flows.

<p align="center">
<img src="../../../../../text_generation_for_gitchat/pic/pic5.bp_through_shortcut.png" width = 55%>
</p>


==References==

1. [waybackprop](https://magenta.tensorflow.org/blog/2017/06/01/waybackprop)

</font>


---

<font size=5>

# Recap the standard LSTM cell


$$\mathbf{f}_t = \sigma (W_f\mathbf{x}_t + U_f\mathbf{h}_{t-1} + \mathbf{b}_f) $$
$$\mathbf{i}_t = \sigma (W_i\mathbf{x}_t + U_i\mathbf{h}_{t-1} + \mathbf{b}_i) $$
$$\mathbf{o}_t = \sigma (W_o\mathbf{x}_t + U_o\mathbf{h}_{t-1} + \mathbf{b}_o) $$
$$\mathbf{\hat{c}}_t = \text{tanh}(W_c\mathbf{x}_t + U_c\mathbf{h}_{t-1} + \mathbf{b}_c) $$
$$\mathbf{c}_t = \mathbf{f}_t \circ \mathbf{c}_{t-1} + \mathbf{i}_t \circ \mathbf{\hat{c}}_t $$
$$\mathbf{h}_t = \mathbf{o}_t \circ \text{tanh}(\mathbf{c}_t) $$

</font>

---

<font size=5>

# Summary: core of LSTM

1. Introduce the cell memory (GRU has no cell).
1. In LSTM, history information flow along with the redline, and no non-linear squashing function is applied.

<p align="center">
<img src="../../../../../text_generation_for_gitchat/pic/pic6.lstm.png">
</p>

</font>

---

<font size=5>

## Question: are there no parallelism at all for RNNs?

1. Stacked multiple RNNs
1. Bidirection

<p align="center">
<img src="images/CudnnLSTM.png"><br>
<font size=4>Fig. As dependencies are resolved a wavefront of operations moves through the network.</font>
</p>


* _This picture is from the blog [Optimizing Recurrent Neural Networks in cuDNN 5](https://devblogs.nvidia.com/optimizing-recurrent-neural-networks-cudnn-5/)_

</font>

---

<!-- *template: invert -->

# ==RNN Variants==

---

<font size=5>

## Dilated RNN (NIPS 2017)

Problem to address

1. complex dependencies in long sequence
1. vanishing and exploding gradients
1. efficient parallelization.

Approach

- multi-resolution dilated recurrent skip connections that can be applied to any RNN cell

</font>

---

<font size=5>

## Dilated RNN (II)

<p align="center">
<img src="../images/DilatedRNN1.png" width=75%>
</p>

<p align="center">
<img src="../images/DilatedRNN.png" width=75%>
</p>

</font>

---

<font size=5>

## Clockwork RNN (CW-RNN, 2014)

- Problem to address
    1. vanishing/gradient.
    1. learn patterns that span at different time scale.
    1. less computation but learning comparable performance as LSTM.

</font>

---

<font size=5>

## Clockwork RNN (I)

1. Partition hidden layer into sub-modules.


<p align="center">
<img src="../images/wh.png">
</p>

</font>

---

<font size=5>

## Clockwork RNN (II)

2. Assign different clock period to different modules.

<p align="center">
<img src="../images/active_modules.png">
</p>

</font>

---

<font size=5>

## Clockwork RNN (III)

3. For unactive modules, directly copy their values in previous time step to current time steps.

<p align="center">
<img src="../images/CWRNN.png">
</p>

</font>

---

<font size=5>

## Downside of Clockwork RNN


How to choose clock period significantly affect clockwork RNN's learning performance and is non-trivial.


</font>

---

<font size=5>

## Multi-dimensional LSTM (MD-LSTM)

- Problem to solve: Applied RNN to multi-dimensional data rather than sequence.

- Models
    1. Replace the single recurrent connection found in standard RNNs with _**as many recurrent connections**_ as there are dimensions in the data.
    1. During the forward pass, at each point in the data sequence, the hidden layer of the network receives both an external input and its activations from one step back along all dimensions.

<p align="center">
<img src="../images/multi-dimensioanl-rnn.png" width=75%><br>
<font size=4>Fig1. Figure 1 and 2 from the paper.</font>
</p>

</font>

---

<font size=5>

## Multi-dimensional LSTM (MD-LSTM)

3. MD-LSTM can also be equipped with directional information.

<p align="center">
<img src="../images/multi-dimensional-multi-directional-context.png" width=75%><br>
Fig2. Figure 4 and 5 from the paper.
</p>

</font>

---

<font size=5>

## MD-LSTM on 2D

<p align="center">
<img src="../images/2d_lstm_1.png" width="35%"><br> LSTM on 2D
</p>

<font size=4>

1. $N$: the dimension number
1. Input $\mathbf{x}$ that is arranged in an $N$-dimensional grid
     - such as 2-D grid of pixels in an image.
1. $N$ hidden vectors $\mathbf{h}_1, ..., \mathbf{h}_N$
1. $N$ memory vectors $\mathbf{c}_1,...,\mathbf{c}_N$

$$\mathbf{c}_t = \sum_{i}^{N}\mathbf{f}_t^i \circ \mathbf{c}^i_t + \mathbf{i}_t \circ \mathbf{\hat{c}}_t$$

</font>
</font>

---

<font size=5>

## Grid LSTM

- Extend LSTM cell to deep networks within a unified architecture.
- Propose a novel robust way for modulating $N$-way communication across the LSTM cells.

</font>

---

<font size=5>

# GridLSTM

_**Inputs**_:

1. a $N$-dimensioanl block receives $N$ hidden vectors: $\mathbf{h}_1, \mathbf{h}_2, ..., \mathbf{h}_N$ and,
1. $N$ memory vectors $\mathbf{m}_1, \mathbf{m}_2, ..., \mathbf{m}_N$

_**Compute**_:

1. deploys cells along _**any**_ or _**all**_ of the dimensions including the depth of the network;
1. _**concatenate**_ all input hiddens to form $\mathbf{H}=[\mathbf{h}_1, ..., \mathbf{h}_N]'$. <span style="background-color:#ACD6FF;">_**This is the difference from HM-LSTM**_.</span>
1. compute $N$ LSTM transforms: $(\mathbf{h}_i, \mathbf{m}_i) = \text{LSTM}(\mathbf{H}, \mathbf{m}_i, \mathbf{W}_i)$ where $i = [1, ..., N]$, $W$ cancatenates $\mathbf{W}_i^i$, $\mathbf{W}_f^i$, $\mathbf{W}_o^i$, $\mathbf{W}_c^i$ in $\mathbb{R}^{d \times Nd}$.

</font>

---


<font size=5>

# GridLSTM

<font size=4>

### Priority Dimensions

1. in general case, a $N$-dimensional block computes the transforms for all dimensions are _**in parallel.**_
1. prioritize the dimension of the network. For dimensions other than prioritized dimensions, their output hidden vectors are computed first, and finally, the prioritized.
    - for example, to prioritize the first dimension of the network, the block first computes the $N$ − 1 transforms for the other dimensions obtaining the output hidden vectors $\mathbf{h}_2', ..., \mathbf{h}_N$.

### Non-LSTM dimensions

Along some dimension, regular connection instead of LSTM is used.

$$\mathbf{h}'_1 = \alpha(\mathbf{V} * \mathbf{H})$$

$\alpha$ above is a standard nonlinear transfer function or identity mapping.

</font>

---

<font size=5>

# An example: a 3D GridLSTM

<p align="center">
<img src="../images/3D-GridLSTM.png">
</p>

</font>


---

<font size=5>

# 3D GridLSTM for NMT

<p align="center">
<img src="../images/GridLSTM-NMT.png" width=45%> <br>Fig. GridLSTM for NMT.
</p>

</font>

---

<font size=5>

## Hierarchical Multiscale Recurrent Neural Networks (HM-RNN)

- Problem to address:
    Learn the hierarchical multiscale structure from temporal data ***without explicit boundary information***.

</font>

---

<font size=5>

# HM-RNN (I)

## Key points of the model

<font size=4>

1.  <span style="background-color:#ACD6FF;">Introduce ***a parametrized binary boundary detector $z_t^i$***</span> at ==**each layer**==.
    - turned on only at the time steps where a segment of the corresponding abstraction level is completely processed.
    - boundary state is a discrete variable.
    - ***straight-through estimator*** is used to calculate gradient of the boundary detector.
1. One of three operations: **UPDATE**, **COPY**, and **FLUSH** is choosen according to $z_t^{l-1}$ and $z_{t-1}^l$

</font>

<p align="center">
<img src="../images/boundary_state.png" width=60%>
</p>
</font>

---

<font size=5>

### HM-RNN: boundary detector and action selection


| $z_{t-1}^l$ <br>left| $z_{t}^{l-1}$<br>buttom | The Selected Operation ||
|:-----------:|:-------------:|:----------------------:|:-:|
|      0      |       0       |        **COPY**        |buttom and left states both do not reach to a boundary.|
|      0      |       1       |       **UPDATE**       |left state does not reaches to a boundary but buttom does.<br>**UPDATE** is executed sparsely|
|      1      |       0       |       **FLUSH**        |left state reaches to a boundary.|
|      1      |       1       |       **FLUSH**        |left state reaches to a boundary.|


<font size=4>

- **UPDATE**: similar to update rule of the LSTM, but executed _**SPARSELY**_.
- **COPY**: ***simply copies***. Upper layer keeps its state unchanged until it receives the summarized input from the lower layer.
- **FLUSH**: executed when a boundary is detected
    - it first ejects the summarized representation of the current segment to the upper layer
    - then reinitializes the states to start processing the next segment.

</font>
</font>

---

<font size=5>

### HM-LSTM: equations

1. pre-activation

    <p align="center">
    <img src="../images/HM-LSTM-pre-activation.png" width=80%>
    </p>

1. cell update

    <p align="center">
    <img src="../images/hm-lstm-cell-update.png" width=65%>
    </p>

1. output hidden

    <p align="center">
    <img src="../images/hm-lstm-output-hidden.png" width=40%>
    </p>

</font>

---

<font size=5>

### STE (Straight-through estimator)

<font size=4>

1. Straight-through estimator.
    - forward pass uses the step function to activate $z_t^l$
    - backward pass uses [hardsigmoid](https://stackoverflow.com/questions/35411194/how-is-hard-sigmoid-defined) function as the biased estimator of the outgoing gradient.
    $$\sigma(x) = \text{max}(0, \text{min}(1, (\alpha x + 1)/2))$$

1. Slope annealing.
    - start from slope $\alpha = 1$.
    - slowly increase the slope until it reaches a threshold. In the paper, the annealing function task-specific.

    <p align="center">
    <img src="../images/hardsigmoid.png" width=45%>
    </p>

</font>
</font>

---

<font size=5>

## ON-LSTM (ICLR 2019 the best paper award)

## The problem to address in this paper

1. natural language is hierarchically structured.
1. design a better architecture _**equipped with an inductive bias towards learning such latent tree structures**_.

</font>

---

<font size=5>

## ON-LSTM: Key ideas

Neurons are split into high-ranking and low-ranking neurons.

|low-ranking neurons|high-ranking neurons|
|:--|:--|
|contains long-term or global information that lasts for several time steps to the entire sentence|encodes local information that lasts only one or a few time steps.

The differentiation between high-ranking and low-ranking neurons is learned in a ==completely data-driven== fashion by controlling the update frequency of single neurons.

1. to erase (or update) high-ranking neurons, the model _**should first erase (or update) all lower-ranking neurons**_.
1. low-ranking neurons are _**always**_ updated more frequently than high-ranking neurons others, and order is pre-determined as part of the model architecture.

</font>

---

<font size=5>

## Recap LSTM

$$\mathbf{f}_t = \sigma (W_f\mathbf{x}_t + U_f\mathbf{h}_{t-1} + \mathbf{b}_f) $$
$$\mathbf{i}_t = \sigma (W_i\mathbf{x}_t + U_i\mathbf{h}_{t-1} + \mathbf{b}_i) $$
$$\mathbf{o}_t = \sigma (W_o\mathbf{x}_t + U_o\mathbf{h}_{t-1} + \mathbf{b}_o) $$
$$\mathbf{\hat{c}}_t = \text{tanh}(W_c\mathbf{x}_t + U_c\mathbf{h}_{t-1} + \mathbf{b}_c) $$
$$\mathbf{h}_t = \mathbf{o}_t \circ \text{tanh}(\mathbf{c}_t) $$

==Modificatioins made to standard LSTM==

1. make input and forget gate for each neuron dependent on others
1. replace the update function for the cell state $\mathbf{c}_t$.

</font>

---

<font size=5>

## ON-LSTM: cumax as gate activation

The `cumax` activation function: $\hat{g}$ which is the cumulative sum of softmax.

$$\mathbf{\hat{g}} = \text{cumsum(softmax(...))}$$


### Note for `cumax`

1. $\mathbf{g} = [0,...,0,1,..., 1]$ is a binary gate that splits the cell into two segments: the 0-segment and the 1-segment. As a result, the model can apply different update rules on the two segments.
1. $\mathbf{\hat{g}}$ is the expectation of the binary gate $\mathbf{\hat{g}} = \mathbb{E}[\mathbb{g}]$.

    - <span style="background-color:#ACD6FF;">ideally, $\mathbf{g}$ should take the form of a _**discrete**_ variable</span>, but computing gradients when a discrete variable is included in the computation graph is not trivial.
    - $\mathbf{\hat{g}}$ here is a continuous relaxation.

</font>

---

<font size=5>

## ON-LSTM: structured gating

Master forget and input gate

$$\tilde{f}_t = \text{cumax}(W_{\tilde{f}}x_t + U_{\tilde{f}}h_{t-1} + b_{\tilde{f}})$$
$$\tilde{i}_t = 1-\text{cumax}(W_{\tilde{i}}x_t + U_{\tilde{i}}h_{t-1} + b_{\tilde{i}})$$

|$\tilde{f}_t$|$\tilde{i}_t$|
|:--|:--|
|controls the erasing behavior|controls the writing behavior|
|monotonically increasing|monotonically decreasing|

<font size=4>

_**NOTES for master gates**_

1. master gates only focus on coarse-grained control.
1. it is computationally expensive and unnecessary to model them with the same dimensions as the hidden states.
1. the paper sets $\tilde{f}_t$ and $\tilde{t}_t$ to bo $D_m = \frac{D}{C}$ where $C$ is the chunk size factor.
1. each dimension of the master gates are repeated $C$ times before the element-wise multiplication with LSTM's original forget gates $f_t$ and input gates $i_t$.

</font>
</font>

---

<font size=5>

## ON-LSTM: cell update

update rules for $c_t$ based on master gates

$$\omega_t = \tilde{f}_t\circ \tilde{i}_t $$
$$\hat{f}_t = f_t \circ \omega_t + (\tilde{f}_t - \omega_t) = \tilde{f}_t \circ (f_t \circ \tilde{i}_t + 1 - \tilde{i}_t) $$
$$\hat{i}_t = i_t \circ \omega_t + (\tilde{i}_t - \omega_t) = \tilde{i}_t \circ (i_t \circ \tilde{f}_t + 1 - \tilde{f}_t) $$
$$c_t = \hat{f}_t \circ c_{t-1} + \hat{i}_t \circ \hat{c}_t $$

$\omega_t$ represents the overlap of $\tilde{f}_t$ and $\tilde{i}_t$

</font>

---

<!-- *template: invert -->

<font size=5>

# ==Summary==


1. RNNs can model data produced by context-free grammars and context-sensitive grammars.
1. Gate is necessary to modulate gradient flow.
1. In practice, RNN models are all stacked to form deep RNN network. The depth seldom accesses 10 layers.

<font size=4>

|RNN Variants|Key Lessons|
|:--:|:--|
|Dilated LSTM|skip connection|
|CW-RNN|split hidden into modules, and modules are updated sparsely|
|MM RNN/Grid LSTM|Recurrent computation to high order tensor|
|Hierarchical Multiscale Recurrent Neural Networks|sparsely updated cell|
|ON-LSTM|split cell into low and high rank parts whose updating frequency are differentiated|

</font>
</font>
