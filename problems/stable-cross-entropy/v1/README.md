# 001 · Why did the loss become infinite?

Machine learning fundamentals · Python / NumPy · 20–25 minutes

A classifier gives the correct class a score of −1000 and the other class a score of 0. Both scores are finite, but the starter returns infinity. Repair cross_entropy to return the mean cross-entropy across the batch. For scores z and correct class y, the per-row loss is −ln(exp(z[y]) / sum(exp(z[j]))). You may rewrite this formula. Use Python and NumPy. logits is a float64 array with shape (N, C), 1 ≤ N ≤ 128, 2 ≤ C ≤ 256 and scores in [−10000, 10000]. labels is an integer array of shape (N,) with a valid class index for each row. Return one finite scalar. Absolute and relative tolerances are both 1e-6.

```python
import numpy as np

def cross_entropy(logits, labels):
    scores = np.exp(logits)
    probs = scores / scores.sum(axis=1, keepdims=True)
    correct_probs = probs[np.arange(len(labels)), labels]
    return -np.log(correct_probs).mean()
```

| Logits | Correct class per row | Expected mean loss |
| --- | --- | ---: |
| `[[0,0]]` | `[0]` | 0.693147 |
| `[[0,-1000]]` | `[1]` | 1000.000000 |
| `[[0,0],[0,-1000]]` | `[0,1]` | 500.346574 |

## Required diagnostic

After the code passes, answer these questions one at a time. Passing the code tests and both questions earns **Solved**.

1. Which intermediate value first rounds to zero in the original starter for logits [[0, -1000]] and labels [1]?
   - A. The exponential of the first class score
   - B. The exponential of the correct class score
   - C. The sum of the exponentials

2. Which operation then produces positive infinity?
   - A. Dividing the zero numerator by the row sum
   - B. Taking the mean of finite losses
   - C. Taking the negative logarithm of the zero probability

## Optional interview follow-up

Derive the gradient of the mean batch loss with respect to each class score on paper. Using NumPy, return a finite array of the same shape as logits. Inputs and tolerances match the forward problem. For reflection, invent a two-row example that catches differentiating the sum instead of the mean. How far off would the answer be?

```python
import numpy as np

def cross_entropy_backward(logits, labels):
    pass
```

For `[[0, 0]]` and label `[1]`, the expected gradient is `[[0.5, -0.5]]`. Increasing the correct class score should reduce the loss.
