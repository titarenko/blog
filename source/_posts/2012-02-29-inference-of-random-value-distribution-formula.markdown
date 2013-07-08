---
layout: post
comments: true
title: "Inference of Random Value Distribution Formula"
categories: Snippet Math Python
---

*This post is just reflection of mind mechanics, which can be useful for ones studying or interested in math.*

Watching Central Limit Theorem lecture (suddenly I enrolled Model Thinking class), I wished to generalize model of coin flipping and get common formula for distribution of random value described below.

So, let's suppose we have `N` things to happen in `1` experiment and we are running `K` experiments. What is distribution of `X` - count of certain single result appearances?

As for me, the best way is to infer formula from simple example. Imagine, we have `3` balls in a box: `"1"`, `"2"`, `"3"`. Next, our experiment looks like this: take a ball, write its number, put it back. Given `4` experiments what results can we have?

Let's write simple script to model our experiment.

```python
from itertools import product, groupby
from operator import itemgetter
c = ["".join(x) for x in product("123", repeat=4)]
d = sorted([(x, x.count("1")) for x in c], key=itemgetter(1))
for x in d:
    print(x[0] + ": " + str(x[1]))
```

So, here are the results (combination itself and count of `"1"`):

```
 2222: 0
 2223: 0
 2232: 0
 2233: 0
 ...
 3333: 0
 1222: 1
 1223: 1
 ...
 1333: 1
 2122: 1
 2123: 1
 ...
 3331: 1
 1122: 2
 1123: 2
 1132: 2
 ...
 3311: 2
 1112: 3
 1113: 3
 1121: 3
 ...
 3111: 3
 1111: 4
```

So, let's start from `0` appearances. In this case combination can be composed from only `2` balls, that is `(N-1)`, since third has no appearances. OK, how much combinations we can compose from `2` balls taken `4` times? Right, `2^4 = 16`.

Next, let's consider case when certain ball appears only once. Here we have slightly more complex case: we still have `2` balls and `3` positions for them, but now we can also insert third ball in the combination - `1xxx`, `x1xx` and so on. Accounting that, there are `C(4,1)` ways to do this insertion, that's why final count of combinations is `C(4,1)*2^3 = 32`.

Now we have `2` appearances. This case is pretty similar to above one, but at the moment we have to pick `2` places for `"1"` and there are `2` places left for `"2"` and `"3"`. So, combinations count is `C(4,2)*2^2 = 24`.

Next case, with `3` appearances of `"1"`, is again very similar to previous ones: we are picking `3` places for `"1"` and we have only one place for `"2"` or `"3"`. Combinations count is `C(4,3)*2 = 8`.

Last case is the simplest one - we can have `4` appearances of `"1"` in `4` experiments only once.

So, looking at our consideration, we can infer common formula: `Q(x) = C(K,x)*(N-1)^(K-x)`.

Let's test this formula with case from Central Limit Theorem lecture. So, we have a coin and 5 flips. Due to nature of process we are expecting normal distribution. Let's verify that: `Q(x) = C(5,x)*(2-1)^(5-x) = C(5,x)`.

So, our formula is correct, giving us next results:

```
Q(0) = C(5,0) = 1
Q(1) = C(5,1) = 5
Q(2) = C(5,2) = 10
Q(3) = C(5,3) = 10
Q(4) = C(5,4) = 5
Q(5) = C(5,5) = 1
```
