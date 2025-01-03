---
sidebar: false
header: false
footer: false
pager: false
---

# Intro to probabilistic programming

## Introduction

If one wants to estimate an unknown quantity of interest, the question of uncertainty arises. 

An example of this can be found in weather forecasts: how likely will it rain today or tomorrow?

What is the best optimal way to assess your uncertainty and best estimate about a given situation ?


## Probability

According to [some](https://en.wikipedia.org/wiki/Edwin_Thompson_Jaynes) [scholars](https://en.wikipedia.org/wiki/Bruno_de_Finetti), probability can be seen as reflecting one's beliefs about some event, involving uncertainty (or certainty). This way, there are only __subjective beliefs__.

Let's say a friend flips a coin, hide the result from me and then peeks at it, and say he sees _Heads_. He is now certain that the __probability__ that this precise coin flip leads _Heads_ is __1__.   
But I am not, and since I didn't receive any other information, I'm going to assign a probability that reflects my belief that the coin is fairly balanced: a probability of __0.5__.

For punters, one can see this probability as the __inverse of the fair [decimal odds](https://en.wikipedia.org/wiki/Odds#Decimal_odds)__ I reckon for this problem.

Suppose my friend then tells me that he has seen this coin before, and that it gave Heads most of the time. Is my belief the same? Rather I am going to __update__ it with this new information.

The __Bayes theorem__ (or formula) tells us exactly _how_ to update one's belief, i. e. update the __implied odds__.

## Probability density

To get bayesian inference, one needs to grasp the concept of probability density. 

It is used to __measure__ a probability. Consider the event that a random variable X is in an interval \[a, b\], i.e. X > a and X < b.

We have to measure the __area under the curve__ (the integral) of the density to obtain the probability of our event.

```js
const lowerBound = view(Inputs.range([0, 1], {
  value: 0.3,
  label: "lower bound",
  format: (x) => x.toFixed(2)
}))
```

```js
const upperBound = view(Inputs.range([0, 1], {
  value: 0.6,
  format: (x) => x.toFixed(2),
  label: "upper bound"
}))
```

```js
Plot.plot({
  marginLeft: 50,
  marks: [
    Plot.line(demoDensity, { x: "x", y: "p" }),
    Plot.areaY(
      demoDensity.filter((d) => (d.x > lowerBound) & (d.x < upperBound)),
      { x: "x", y: "p", fill: "lightblue" }
    )
  ]
})
```

```js
const mass = demoDensity
  .filter((d) => (d.x > lowerBound) & (d.x < upperBound))
  .reduce((s, x) => s + x.p, 0)
  .toFixed(3)
```

```js
Plot.plot({
  marks: [
    Plot.barX([{ mass: mass, total: 1 }], {
      x: "mass",
      y: "total",
      fill: "lightblue"
    }),
    Plot.barX([1], { fill: "lightgrey", fillOpacity: 0.4 })
  ],
  x: {
    domain: [0, 1]
  }
})
```

## Bayesian inference

In __bayesian inference__, we mostly pay attention to the numerator of Bayes' formula for probability distributions.

Indeed, we have one set of __Data__ (D), and __unobserved parameters__ (theta).

We want to estimate the posterior probability distribution of the unobserved parameters given the (observed) data.

Bayes' theorem states the following:

```tex
p(\theta|\mathcal{D}) \propto {p(\mathcal{D}|\theta) \times p(\theta)}
```

```tex
\mathrm{posterior} \propto \mathrm{likelihood} \times \mathrm{prior} 
```

To get rid of the constant, we can multiply the two terms, and normalize the result so that the posterior sums (integrate) to 1 across all possible theta values.

In practice we use log-probabilities, so the above equation turns into a sum of the type `log-posterior = log-likelihood + log-prior + C` where C is a constant independent of theta.

## Coin tossing

Let's flip a coin. We don't know wheter the coin is biased. We can only toss it and observe the outcome.

Denote p the probability of heads. We want to estimate this quantity.

```tex
0 \le p \le 1
```

**Slide** the quantity **below** to toss the coin `nTosses` times. We then show
- Number of heads and Tails
- Mode of posterior and 95% credible interval with time

```js
const nTossesInput = Inputs.range([0, totalTosses], {
  step: 1,
  label: "nTosses",
  value: 5
})
const nTosses = Generators.input(nTossesInput)
```

<div>${nTossesInput}</div>

```js
Plot.plot({
  marginLeft: 70,
  grid: false,
  y: {
    label: "Side"
  },
  marks: [
    Plot.ruleX([0]),
    Plot.tickX(ticks, { x: "flip", y: "coin", strokeWidth: 1 })
  ]
})
```

```js
Plot.plot({
  color: {
    scheme: "accent"
  },
  marks: [
    Plot.barY(
      [
        { side: "Heads", count: nbHeads },
        { side: "Tails", count: nTosses - nbHeads }
      ],
      { x: "side", y: "count", fill: "side" }
    )
  ],
  y: {
    domain: [0, nTosses]
  }
})
```

```js
Plot.plot({
  y: {
    grid: true,
    domain: [0, 1],
    label: "↑ Coin bias"
  },
  x: {
    label: "→ Tosses"
  },
  marks: [
    Plot.line(cumul, { x: "total", y: "proportionH" }),
    Plot.line(
      firstModes.map((x, i) => ({ mode: x, nTosses: i })),
      { x: "nTosses", y: "mode", stroke: "blue" }
    ),
    Plot.ruleY([truep], { stroke: "black", strokeDasharray: [5] }),
    Plot.areaY(
      selectedQ.q05
        .slice(0, nTosses)
        .map((x, i) => ({ q05: x, q95: selectedQ.q95[i], toss: i })),
      {
        y1: "q05",
        y2: "q95",
        fill: "lightblue",
        fillOpacity: 0.7
      }
    )
  ]
})
```

```js
Plot.plot({
  marginLeft: 70,
  x: {
    label: null
  },
  width: 120,
  color: {
    range: ["black", "lightblue"]
  },
  marks: [
    Plot.cell(
      [
        { prior: "uniform", color: "1" },
        { prior: "selected", color: "2" }
      ],
      { y: "prior", fill: "color" }
    )
  ]
})
```

```js
Inputs.bind(
  Inputs.range([0, totalTosses], { step: 1, label: "nTosses", value: 1 }),
  nTossesInput
)
```

You can also change the number of tosses above, and **select a prior below**. Watch how the prior changes the posterior distribution.

The largest differences can be found with a **low** amount of tosses, while the posterior converges most of the time towards the same limit distribution.

Note that to be _fully bayesian (TM)_, you probably need to take the mean of the posterior distribution rather than the mode (value with the highest probability).

```js
const priorName = view(Inputs.radio(allPriorNames, {
  label: "Prior",
  value: "fair"
}))
```

```js
Plot.plot({
  y: {
    grid: true,
    label: "↑ Probability density"
  },
  x: {
    label: "→ Coin bias"
  },
  marks: [
    Plot.line(likelihoodNorm, { x: "p", y: "prob", stroke: "red" }),
    Plot.line(
      grid.map((x, i) => ({ p: x, prob: prior[i] })),
      { x: "p", y: "prob", stroke: "orange" }
    ),
    Plot.line(
      grid.map((x, i) => ({ p: x, prob: posterior[i] })),
      { x: "p", y: "prob", stroke: "darkblue" }
    ),
    Plot.areaY(
      grid
        .map((x, i) => ({ p: x, prob: posterior[i] }))
        .filter((x, i) => cdf[i] > 0.05 && cdf[i] < 0.95),
      { x: "p", y: "prob", fill: "lightblue", fillOpacity: 0.7 }
    ),
    Plot.ruleX([truep], { stroke: "black", strokeDasharray: [5] }),
    Plot.ruleX([grid[argMax(posterior)]], { stroke: "blue" })
  ]
})
```

```js
Plot.plot({
  marginLeft: 70,
  x: {
    // label: null
  },
  width: 130,
  height: 100,
  color: {
    range: ["orange", "red", "lightblue"]
  },
  marks: [
    Plot.cell(
      [
        { density: "prior", color: "1" },
        { density: "likelihood", color: "2" },
        { density: "posterior", color: "3" }
      ],
      { y: "density", fill: "color" }
    )
  ]
})
```

```js
const mode = grid[argMax(posterior)]
```

## Let's take bets

Supposed you haven't seen the dashed line, and that you witnessed some tosses. Now, I toss it one more time. 

> What are the __fair decimal odds__ that you reckon to bet that the coin toss leads to _Heads_:
> - after 5 tosses?
> - 10 tosses?
> - 100 tosses?
> - 400 tosses?

```js
const fairOdds = (1 / mode).toFixed(2)
```

## Probabilistic programming

So far, we've done mostly hand-crafted (with my hand) brute-force __bayesian inference__. 

Probabilistic programming languages (PPLs) allow us to specify a model in a specific domain-specific-language (DSL), and the inference engine takes care of the hard work.

For instance, in __Numpyro__, a new PPL based on [JAX](https://github.com/google/jax) in Python, you can specify the above problem as:

```python
import numpyro as ny
import numpyro.distributions as dist


def model(N, obs=None):
  p = ny.sample("p", dist.Normal(0.5, 0.15))
  ny.sample("count_heads", dist.Binomial(N, p), obs=obs)
```

Then you only specify the automatic inference algorithm, and that's it!

In our case, the problem is uni-dimensional: we only have one parameter to identify. 

In real-world™ scenarios, there are often dozens of parameters to identify. It becomes computationally impossible to perform brute-force inference on all of them, that's why smart approximation algorithms such as __Markov Chain Monte Carlo__ (__MCMC__) are so useful. 

The [best ones](https://elevanth.org/blog/2017/11/28/build-a-better-markov-chain/), such as HMC and its variant NUTS, leverage [automatic differenciation](https://en.wikipedia.org/wiki/Automatic_differentiation) (AD). For a small overview of automatic differenciation in French, see [this post](https://techblog.deepki.com/autodiff/).

## To go further
- Jake VanderPlas' talk at SciPy 2014, [Frequentism and Bayesianism: What's the Big Deal?](https://youtu.be/KhAUfqhLakw)

- Cam Davidson-Pilon's best-seller, [Bayesian Methods for Hackers](http://camdavidsonpilon.github.io/Probabilistic-Programming-and-Bayesian-Methods-for-Hackers/), fully open-source and readable online.

- Richard McElreath's most recent book, [Statistical Rethinking](https://xcelab.net/rm/statistical-rethinking/). NB: many code translations are available in Python PPLs.

- See [Cox's theorem](https://en.wikipedia.org/wiki/Cox%27s_theorem):
> Probability is interpreted as a formal system of logic, the natural extension of Aristotelian logic (in which every statement is either true or false) into the realm of reasoning in the presence of uncertainty.


- For probability and bets, see [my blog post](https://horaceguy.pages.dev/posts/betting-theory/) and literature by De Finetti.

- For an application of probabilistic programming to Covid-19 epidemiology and lockdown measures, see the [story](https://horaceguy.pages.dev/posts/uncover-covid/) of my participation to a Kaggle Challenge.

```js
const demoDensity = makeNormalPrior().map((p, i) => ({ x: grid[i], p: p }))
```

```js
function registerModes(grid, samples) {
  let a = samples
    // .slice(0, nTosses)
    .map((_, i) => samples.slice(0, i))
    .map((a) => ({
      count: countHeads(a),
      total: a.length,
      proportionH: countHeads(a) / a.length
    }));

  let likelihoodsNorm = a
    .map((x) => makeLikelihood(grid, x.total, x.count))
    .map(stdzLikelihood);

  let posteriors = allPriorNames.map((name) => ({
    name: name,
    posteriors: likelihoodsNorm.map((x) => makePosterior(x, selectPrior(name)))
  }));

  let cdfs = posteriors.map((d) => d.posteriors.map((post) => makeCDF(post)));

  let data = posteriors.map((d) => ({
    name: d.name,
    mode: d.posteriors.map((post) => grid[argMax(post)]),
    cdfs: d.posteriors.map((post) => makeCDF(post)),
    posteriors: d.posteriors
  }));

  return data.map((d) => ({
    name: d.name,
    mode: d.mode,
    q05: d.posteriors.map(
      (post, i) => grid.filter((x, ii) => d.cdfs[i][ii] > 0.05)[0]
    ),
    q95: d.posteriors.map(
      (post, i) => grid.filter((x, ii) => d.cdfs[i][ii] < 0.95).reverse()[0]
    )
  }));
}
```

```js
const modes = registerModes(grid, samples)
```

```js
const firstModes = modes.filter((d) => d.name == priorName)[0].mode.slice(0, nTosses)
```

```js
const selectedQ = modes.filter((d) => d.name == priorName)[0]
```

```js
const toss = (p) => {
  if (Math.random() < p) {
    return "Heads";
  } else {
    return "Tails";
  }
}
```

```js
const totalTosses = 150
```

```js
const countHeads = (a) =>
  a.reduce((acc, toss) => (toss === "Heads" ? acc + 1 : acc), 0)
```

```js
const nbHeads = countHeads(samples.slice(0, nTosses))
```

```js
const proportionH = nbHeads / nTosses
```

```js
const samples = Array(totalTosses)
  .fill()
  .map((x) => toss(truep))
```

```js
const ticks = samples.slice(0, nTosses).map((x, i) => ({ flip: i, coin: x }))
```

```js
const cumul = samples
  .slice(0, nTosses)
  .map((x, i) => samples.slice(0, i))
  .map((a) => ({
    count: countHeads(a),
    total: a.length,
    proportionH: countHeads(a) / a.length
  }))
```

```js
const grid = Array(nGrid)
  .fill()
  .map((x, i) => i / nGrid)
```

```js
const allPriorNames = [
  "uniform",
  "fair",
  "half",
  "badNormal",
  "fairNormal",
  "goodNormal"
]
```

```js
const uniformPrior = grid.map((_) => 1 / nGrid)
```

```js
const halfPrior = grid.map((x, i) => (i < nGrid / 2 ? 0 : 2 / nGrid))
```

```js
function makeSawtooth() {
  let tooth = grid.map((x, i) => grid.length - i);
  return tooth.map((x) => x / tooth.reduce((s, x) => s + x, 0));
}
```

```js
const sawtoothPrior = makeSawtooth()
```

```js
function makeFairPrior() {
  let unstd = grid.map((x, i) => (x * (1 - x)) / 4);
  return unstd.map((x) => x / unstd.reduce((s, x) => s + x, 0));
}
```

```js
const fairPrior = makeFairPrior()
```

```js
const normalDensity = (mu, sigma, x) =>
  (1 / (sigma * Math.sqrt(2 * Math.PI))) *
  Math.exp(-(((x - mu) / sigma) ** 2) / 2)
```

```js
function normal(mu, sigma, grid) {
  return grid.map((x) => normalDensity(mu, sigma, x));
}
```

```js
function makeGoodPrior() {
  let unstd = normal(truep, 0.05, grid);
  return unstd.map((x) => x / unstd.reduce((s, x) => s + x, 0));
}
```

```js
function makeNormalPrior() {
  let unstd = normal(0.5, 0.15, grid);
  return unstd.map((x) => x / unstd.reduce((s, x) => s + x, 0));
}
```

```js
const normalPrior = makeNormalPrior()
```

```js
function makeBadPrior() {
  let unstd = normal(truep * 2, 0.05, grid);
  return unstd.map((x) => x / unstd.reduce((s, x) => s + x, 0));
}
```

```js
const badNormalPrior = makeBadPrior()
```

```js
function selectPrior(priorName) {
  switch (priorName) {
    case "uniform":
      return uniformPrior;
    case "half":
      return halfPrior;
    case "sawtooth":
      return sawtoothPrior;
    case "fair":
      return fairPrior;
    case "goodNormal":
      return goodNormalPrior;
    case "fairNormal":
      return normalPrior;
    case "badNormal":
      return badNormalPrior;
  }
}
```

```js
const prior = selectPrior(priorName)
```

```js
const truep = 0.33
```

```js
const Cnk = (n, k) => Math.exp(logCnk(n, k))
```

```js
function logCnk(n, k) {
  let logCoef = Array(k)
    .fill()
    .map((x, i) => Math.log(n - i) - Math.log(i + 1))
    .reduce((s, x) => s + x, 0);
  return logCoef;
}
```

```js
function coerce(x, repl) {
  if (isNaN(x)) {
    return repl;
  } else {
    return x;
  }
}
```

```js
function binomLogProb(p, n, k) {
  let logp = Math.log(p);
  let log1mp = Math.log(1 - p);
  return logCnk(n, k) + k * logp + (n - k) * log1mp;
}
```

```js
function binomProb(p, n, k) {
  let val = Math.exp(binomLogProb(p, n, k));
  if (val === -Infinity || isNaN(val)) {
    return 0;
  } else {
    return val;
  }
}
```

```js
const likelihood = makeLikelihood(grid, nTosses, nbHeads)
```

```js
const likelihoodNorm = stdzLikelihood(likelihood)
```

```js
function makeLikelihood(grid, nTosses, nbHeads) {
  let likelihood = grid.map((x) => ({
    p: x,
    prob: binomProb(x, nTosses, nbHeads)
  }));

  return likelihood;
}
```

```js
function stdzLikelihood(unstdLikelihood) {
  let likelihoodNorm = unstdLikelihood.map((x, i) => ({
    p: x.p,
    prob:
      x.prob /
      unstdLikelihood.reduce((s, u) => s + (isNaN(u.prob) ? 0 : u.prob), 0)
  }));
  return likelihoodNorm;
}
```

```js
function makePosterior(likelihood, prior) {
  let unstdPosterior = likelihood.map((x, i) => x.prob * prior[i]);
  let posterior = unstdPosterior.map(
    (x) => x / unstdPosterior.reduce((s, x) => s + x, 0)
  );
  return posterior;
}
```

```js
const posterior = makePosterior(likelihood, prior)
```

```js
const cdf = makeCDF(posterior)
```

```js
const goodNormalPrior = makeGoodPrior()
```

```js
function makeCDF(posterior) {
  let cdf = [];
  function push(s, x) {
    let res = s + x;
    cdf.push(res);
    return res;
  }
  posterior.reduce(push, 0);
  return cdf;
}
```

```js
function argMax(array) {
  return array.map((x, i) => [x, i]).reduce((r, a) => (a[0] > r[0] ? a : r))[1];
}
```

```js
const nGrid = 500
```

% iFrame

```js
const observer = new ResizeObserver(([entry]) => parent.postMessage({height: entry.target.offsetHeight}, "*"));
observer.observe(document.documentElement);
invalidation.then(() => observer.disconnect());
```
