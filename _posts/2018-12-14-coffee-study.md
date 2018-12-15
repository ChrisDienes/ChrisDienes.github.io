---
layout: default_post
title: Coffee, Coworkers, and the EM Algorithm
date: 2018-12-14
description: Analysis of a personal coffee consumption data set, with some coworker inferences tossed in.
image: /images/coffeeStudy/ccemalgo.svg
tags: [DIY Project, R]
---

### Setup
  
My office has an honor system for refilling the coffee pot: <em>"If you get the last cup, then you start the next pot"</em>. There's no warning. Our coffee dispensers are 2.5 liter, cylindrical, metal airpots which come devoid of windows. The only indicator you've lost coffee roulette is the announcement from the sputtering air sounds caused by your hopeless pumps of the last few drops. About six months ago I found an airpot empty upon my arrival. A coworker witnessing my fate said: <em>"People suck."</em> As a data scientist I couldn't resist. After consuming 202 cups of data, I can boldly claim that people suck...err...or maybe not.

The experiment is simple. For each coffee trip I recorded some basic information: date, time, and whether or not I was able to fill my 8oz cup. I decided to limit the experiment to the medium roast blend which seems to be the most popular choice at my work. The other options, which include dark roast and some flavored garbage, are presumably the alternatives for anyone who disregards their refilling responsibilities. I also decided to visit the coffee station which was closest to my cube, thus limiting the generalizability of the results. Lastly, whenever I encountered an empty state I ensured I was a good office citizen and refilled the airpot.

I observed 43 (21.3%) refill experiences in my 202 coffee visits. According to the owner's manual, our coffee maker is adjusted at the factory to fill a standard 2.5 liter airpot with 2.2 liters (or 74.4oz) of brewed coffee. Using my 8oz cup with a simple uniform distribution would suggest a refill risk of only 10.75%. Extrapolating the increase in my observed refill rate over the course of an entire year (2 cups a day for 250 days), it appears I'm on track for 53 more refills than my cup size would expect. These numbers seem to suggest some of my coworkers are <em>"delinquent"</em> in their refilling duties. But can we do better and actually estimate how likely it is for a random coworker to be delinquent based on the data collected?

### The Analysis

Let's jump right into the results. I've placed the technical details at the end for those who aren't adverse to math/stats. Keeping it high level, I created a probabilistic model for estimating the probability a coworker is delinquent when they encounter an empty airpot. The model assumes the number of non-empty cups/arrivals in an airpot is fixed. For example, our airpot would be empty about every 9th arrival if everyone used an 8oz cup. I've summarized the probability estimates for various sizes in the below table to account for potential differences in cup volumes. The estimated delinquency rates decrease as the typical cup size increases. On the extremes, estimated delinquency could be as high as 61.75% if everyone used an 8oz cup, and as low as 0% for 20oz barrels. The table also includes 95% (bootstrap) confidence intervals. These intervals appear wide which implies a high degree of uncertainty in our estimation. Since we have a lack of data from other users, perhaps a reasonable assumption for cup size would be somewhere in the middle. This <a href="http://www.e-importz.com/coffee-statistics.php" target="_blank">website</a> seems to suggest the average size is 9oz, so we'll use this scenario throughout the remaining analysis.      

<div style = "text-align:center;overflow-x:auto;">
     <table style="margin: 0 auto;border-collapse:collapse;width: 100%;text-align:center;">
      <tr>
        <th>Cup Size</th>
        <th>Arrivals Till Empty</th>
        <th>Delinquent Percent</th>
        <th>95% CI</th>
      </tr>
      <tr>
        <td>8oz</td>
        <td>9</td>
        <td>61.75</td>
        <td>(36, 76)</td>
      </tr>
      <tr>
        <td>9oz</td>
        <td>8</td>
        <td>54.87</td>
        <td>(25, 71)</td>
      </tr>
      <tr>
        <td>10oz</td>
        <td>7</td>
        <td>45.40</td>
        <td>(10, 65)</td>
      </tr>
      <tr>
        <td>12oz</td>
        <td>6</td>
        <td>31.57</td>
        <td>(0, 56)</td>
      </tr>
      <tr>
        <td>16oz</td>
        <td>5</td>
        <td>9.50</td>
        <td>(0, 41)</td>
      </tr> 
      <tr>
        <td>20oz</td>
        <td>4</td>
        <td>0.00</td>
        <td>(0, 14)</td>
      </tr> 
     </table>
</div>


The below image displays my refill risk and the delinquency rates when partitioning the data by the days of the week. There's a slight trend which increases as we move from Mondays to Thursdays, and then a steep drop-off on Fridays. Perhaps there are highly delinquent individuals who are commonly out-of-office on Fridays, or maybe the end of the work week brings out the best in us. The slight increase over the first four work days seems strange. Maybe there is moral licensing at play: <em>"I've already refilled once this week, so it's okay to let someone else get this one."</em> 

<p align="center">
  <img src="/images/coffeeStudy/dayOfWeek.svg" alt = "image should be here" />
</p>

It's hard to ignore the obvious confounding time of day variable. Presented below is the (smoothed) distribution of my coffee times by hour of day. Obviously, I prefer a cup first thing, followed by a more irregular sampling. The next plot shows my estimated refill risk across time. Three features stand out. The first is a peak in rates between 9am-10am in the morning. It's hard to identify what's really happening. On the one hand, a rush of delinquent employees might arrive during this interval; but on the other, it could simply be a natural time for the airpot to reach its empty state. This identifiability problem seems to limit the pureness in our inferences. Though I will add, whenever I failed to fill my cup completely, I would top it off using one of the other two blends (dark roast or flavored). Several trips I experienced empty airpots for all three blends, with times 9:11am, 9:32am, and 2:04pm. The second interesting feature is the noticeable rise in rates which occur post lunch time. It seems the local coffee drinkers believe in a "cutoff" time; which once past, not refilling is justified as avoiding waste. This "cutoff" time debate appears to become more one-sided as we move further away from twelve of the clock. The last feature to note is the brief time periods where the estimated delinquency rates fall to zero. The model (and data) is suggesting our best guess for these periods is a rate of zero. I think leaving it at that ought to make a few Bayesians twitch.       

<p align="center">
  <img src="/images/coffeeStudy/myArrivals.svg" alt = "image should be here" />
</p>

<p align="center">
  <img src="/images/coffeeStudy/riskByTime.svg" alt = "image should be here" />
</p>

The last curious finding is presented in the below figure which contains the estimated rates relative to the number of minutes from the nearest half hour. The plot shows our rates increase as we approach the top or bottom of the hour, and then decline as we move past. Nearly all meetings have start times with :00 or :30 endings. At times, it's possible the pressure to be on time outweighs the pressure to refill.         

<p align="center">
  <img src="/images/coffeeStudy/riskByHalfHour.svg" alt = "image should be here" />
</p>

### Concluding Thoughts

It's not entirely clear what portion of my misfortunes are caused by coworker delinquency and what portion is related to other factors; such as cup sizes and timing. We would need a more intrusive data sample to more effectively resolve those questions; not something I'm willing to brew.     

### Where the Math-Magic Happens

Let's denote the probability a random coworker is delinquent when encountering an empty airpot by $\rho$. It would be interesting to estimate $\rho$ based on the data collected. However, matters are complicated by the fact we only observe my actions and outcomes, and nothing directly about by my coworkers. In the perfect world we would have all the data. Under this setting, we would describe the experiment using a probabilistic model and choose the probability value $\rho$ which maximizes the likelihood of observing the collected data. This classical statistics problem is known as <a href="https://en.wikipedia.org/wiki/Maximum_likelihood_estimation" target="_blank">maximum likelihood estimation</a> and typically involves some simple calculus. Fortunately, we can still follow this basic outline for our incomplete data set, but we will need to make several assumptions and change the way we optimize for $\rho$.

The primary assumption we'll make is the following: my refill (or non-refill) probability is determined by the number of arrivals which encountered an empty (or non-empty) state for the airpot from which we sampled. This could be similarly stated as the probabilities are proportional to the time spent in a given empty/non-empty state for the sampled airpot, where the time between arrivals is some fixed constant. To see the consequences of this assumption consider the following two sequences, where $(n,d,r)$ represents the arrival outcomes non-empty, delinquent, and refilled:

$$\{n,n,n,n,n,n,n,n,r\};$$
$$\{n,n,n,n,n,n,n,n,d,d,d,r\}.$$

Both of the sequences assume our airpot requires a refill on the 9th arrival. In the first sequence our assumption would imply a refill risk of 11% (1 out of 9). This risk jumps to 33% (4 out of 12) under the second sequence due to the increased time spent in an empty state caused by the streak of three delinquent coworkers. 

Note that we don't actually observe the number of delinquencies per airpot in our data set (denote this number by $z$). We only measure the indicator of whether I had a refill ($x = 1$) or non-refill ($x = 0$) arrival. Thus, conditional on the number of delinquencies, our refill outcome follows: 

$$(X|Z = z) \sim \mbox{Binomial}\left[n = 1, prob = \frac{1 + z}{c + z}\right];$$

where we define $c$ to be the number airpot arrivals until a refill is required. To derive the marginal distribution for the number of delinquencies, first consider the probability of a single delinquency. A delinquency can't occur if I observe an empty state, since by design I'll refill with probability 1. So let's assume my arrivals occur with some small probability $\pi$. For the study results I used $\pi = 0.05$ (or 1 out of 20 cups), and using a larger (smaller) $\pi$ value slightly increases (decreases) our estimates of $\rho$. Hence, the probability of a single delinquency during an empty state is $\tilde{p} = (1 - \pi)*\rho$. Thus, the number of delinquencies prior to the first refill is described by the negative binomial model:

$$\mbox{Pr}[Z = z] = (1 - \tilde{p})\tilde{p}^z, \quad z \in \{0,1,\ldots\}.$$

If you haven't noticed already, I'm assuming all arrivals (including mine) are independent and the probability of delinquency is constant for the entire coworker population. The constant probability assumption is used to simplify our notation and doesn't have any real effect on the estimation. To see this, try constructing the moment functions for the constant $\rho$ probability model:  

$$Z \sim \mbox{Binomial}[n = 1, prob = \rho];$$ 

and the following two step sampling model where on average we get probability $\rho$: 

$$(Z|\phi) \sim \mbox{Binomial}[n = 1, prob = \phi], \quad \mbox{E}[\phi] = \rho.$$

The independence claim may have some influence on the results. In the case of my same day arrivals, on average they were separated by over 3 hours and had a minimum gap of 48 minutes. 

We can now write the full likelihood using our conditional refill and marginal delinquency distributions:

$$L[\rho;X,Z,\pi,c] = \prod_{i = 1}^{202} \left[ \frac{c - 1}{c + z_i}\right]^{1-x_i}\left[\frac{1+z_i}{c + z_i}\right]^{x_i}\left[1 - (1-\pi)\rho\right]\left[(1-\pi)\rho\right]^{z_i};$$

and the full log-likelihood:

$$l[\rho;X,Z,\pi,c] = \sum_{i = 1}^{202}\left[h(x_i,z_i,\pi,c) + z_i\log(\rho) + \log(1 - (1-\pi)\rho)\right];$$

where $h(\cdot)$ does not depend on $\rho$. We wish to maximize $l[\rho;X,Z,\pi,c]$ with respect to $\rho$ but we are missing the $z_i$. The solution is to use the iterative procedure known as the <a href="https://en.wikipedia.org/wiki/Expectation-maximization_algorithm" target="_blank">Expectation Maximization Algorithm</a>, or EM Algorithm. The process will go as follows:

<strong>Step 1.</strong> Initialize a probability value $\rho_0$ at iteration $t = 0$.<br>
<strong>Step 2.</strong> Find the conditional distribution of $(Z|X,\rho_t)$.<br>
<strong>Step 3.</strong> (Expectation Step) Replace the missing $z_i$ with their current conditional expectations:

$$Q(\rho|\rho_t) = \mbox{E}_{Z|X,\rho_t}(l[\rho;X,Z,\pi,c]).$$
 
<strong>Step 4.</strong> (Maximization Step) Set $\rho_{t+1} = \arg\max_\rho Q(\rho|\rho_t)$.<br> 
<strong>Step 5.</strong> Repeat Steps 2-4 until convergence is met.<br>
The above steps ensure we monotonically approach a (local) maximum. Adjusting the initial value in Step 1 can help discover discrepancies between local and global extreme values.     

For our implementation, Step 2 requires numerical approximation of the probabilities:

$$\mbox{Pr}[Z = z | X = x, \rho_t] = \frac{\mbox{Pr}[X = x, Z = z|\rho_t]}{\sum_{z = 0}^{\infty}\mbox{Pr}[X = x, Z = z|\rho_t]}, \quad x = 0,1.$$

The above probabilities were computed and stored for $z = 0,1,\ldots,100$, then used in the EM steps. 

Focusing only on the terms in $Q(\rho|\rho_t)$ which depend on $\rho$, the EM steps become:<br>
<strong>(E Step)</strong> Compute:

$$m_i = \sum_{z=0}^{100}z\cdot\mbox{Pr}[Z = z | X = i, \rho_t], \quad i = 0,1.$$

<strong>(M Step)</strong> Update:

$$\rho_{t+1} = \frac{m_0 n_0 + m_1 n_1}{202 + m_0 n_0 + m_1 n_1}\cdot\frac{1}{1-\pi};$$

where $n_0$ and $n_1$ are the number of observed non-refills and refills, respectively. Obviously, we could further generalize the provided expressions by substituting $n$ for the observed sample size $202$.

### Code

The implementation of the algorithm, data, and analysis code can be found in my GitHub repo <a href="https://github.com/ChrisDienes/CoffeeStudy" target="_blank">here</a>.  

<br>
<br>
