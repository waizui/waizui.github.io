<head>
    <script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
    <script type="text/x-mathjax-config">
        MathJax.Hub.Config({
            tex2jax: {
            skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
            inlineMath: [['$','$']]
            }
        });
    </script>
</head>

# Let's build a GPT-2 (II)

In part I, I explained how **embedding** works. In this part, let's delve into how Attention mechanism works.

## Attention is All You Need

The foundation that powers transformers is **Scaled Dot-Product Attention**, which is:

$$
\text{Attention}(Q,K,V)

=

\text{softmax}
\left(
\frac{QK^T}{\sqrt{d_k}}
\right)
V
$$

where $Q,K,V$ is **Query, Key, and Value** respectively.


```python
class SelfAttention(nn.Module):
    def __init__(self, d_in, d_out, qkv_bias=False) -> None:
        super().__init__()
        # replace this with nn.Parameter for bias operation and optimized initial weights
        self.W_q = nn.Linear(d_in, d_out, bias=qkv_bias)
        self.W_k = nn.Linear(d_in, d_out, bias=qkv_bias)
        self.W_v = nn.Linear(d_in, d_out, bias=qkv_bias)

    def forward(self, x: Tensor):
        q = self.W_q(x)  # [N,d_out] , x @ W_q
        k = self.W_k(x)
        v = self.W_v(x)

        atten_scorce = (
            q @ k.T
        )  # [N,d_out] @ [d_out,N], meaning: [i][j] = ith token's attention to jth token
        atten = softmax(atten_scorce / k.shape[-1] ** 0.5, dim=-1)
        context_vec = atten @ v  # [N, d_out]
        return context_vec
```



## References

- [Attention Is All You Need](https://arxiv.org/abs/1706.03762)
