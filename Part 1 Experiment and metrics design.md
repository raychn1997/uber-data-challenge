# Part 1 Experiment and metrics design

# Context

The Driver Experience team has just finished **redesigning 
the Uber Partner app**. The new version expands the purpose of the app beyond just driving.
It includes additional information on earnings, ratings, 
and provides a unified platform for Uber to communicate with its partners.

# Questions

1. Propose and define **the primary success metric** of the redesigned app. 
What are 2- 3 additional tracking metrics that will be important to monitor in 
addition to the success metric defined above?

As a business, Uber generates revenues by taking commissions from each ride.
Therefore, assume the average commission per ride does not change, we can safely say that
the more rides it has, the more money it makes, which gives us the north star metric:

- **Revenue Metric: average number of rides per day**

However, this metric can be further broken down into 2 components: *the number of active drivers * the average number of rides per driver*.

The increase in the first metric would signal **growth**(getting new users). 
Likewise, the increase in the second would signal **more engagement**(drivers spend more time on the app). 
Therefore, the additional metrics can be:

- **Growth Metric: number of active drivers(drivers who have at least 1 ride per day on average)**
- **Engagement Metric: average number of rides per active driver per day**

It is important to set a threshold and define active drivers rather than just use the number of drivers
(The threshold should probably be based on the judgement of the product manager and 1 is just an example),
because the latter is just a **vanity metric**, and, if drivers sign up to the app just due to the **novelty effect**, 
and they stop using the app after it wears off, Uber is not going to earn money from them. 
So, we need only active drivers.

2. Outline **a testing plan** to evaluate if redesigned app performs better 
(according to the metrics you outlined). How would you balance the need
to deliver **quick results**, with **statistical rigor**, and while still **monitoring for risks**?

Assume we want to do a test with
- **a significance level of 0.05** (the probability of switching to the new app when it is not actually better)
- **a power of 0.80** (the probability of switching to the new app when it IS actually better)
- **a minimal effect size of 0.05** (the increase in the average ride number per day that we deem worthy of the roll-out)

The first two are pretty much the industrial standards. However, the last one is subject to the judgement of the product
manager; 0.05 is just my educated guess. However, according to Google, **Uber has an average ride of 7.5** and 
its annual revenue in 2020 is 11.1 billion dollars, which means a 0.05 increase would translate to `0.05/75*11.1 = 0.0074`
billion dollars, a good amount of money.

Here we also assume a **standard deviation of 1**(which should be calculated from the past data) and use a **two-sided T test**,
from which we can work out the **sample size** to be **6280**. 

```python
from statsmodels.stats.power import TTestIndPower
minimal_effect = 0.05
standard_deviation = 1
effective_size = minimal_effect/standard_deviation
n = TTestIndPower().solve_power(power = 0.8, alpha = 0.05, effect_size=effective_size)
print('{} drivers are needed per sample'.format(int(n)))
```

Now that all the math has been worked out, we can define a testing plan:
- Find 1 pair of cities that are comparable to each other in terms of demographics and behavior data.
Both cities should have similar performance across different metrics prior to the test. They should also have
**at least 6280 active Uber drivers each**.
- Launch the new version in city A and gather the metrics for **two weeks**.
- Analyze the test result

To ensure statistical rigor, even if we have a pair of mega-cities with one million drivers in each, we should still
run the test for at least 2 weeks to ensure, at least, that weekday seasonality and novelty effect doesn't bias our result.

3. Explain how you would **translate the results from the testing plan 
into a decision** on whether to launch the new design or roll it back.

If the revenue metric rises in a statistically significant way, we launch the new product;
otherwise, if the revenue metric doesn't increase, or even decreases significantly, we look into the two monitoring metrics.

We dig into the data and check whether it's the engagement or growth metric that is under-performing, and whether the under-performance is across
all the groups (language, part-time vs full time, etc.) so that we can **develop new hypotheses**(like bugs or confusing UI) for possible improvement
ideas(**and conduct new AB tests on them**).
If there does not seem to be a quick/inexpensive fix, we may consider rolling back the new app or even discarding it all together.
