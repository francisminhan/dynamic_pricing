# Dynamic Pricing

## Problem
You’re launching a ride-hailing service that matches riders with drivers for trips between the Toledo Airport and Downtown Toledo. It’ll be active for only 12 months. You’ve been forced to charge riders $30 for each ride. You can pay drivers what you choose for each individual ride.

The supply pool (“drivers”) is very deep. When a ride is requested, a very large pool of drivers see a notification informing them of the request. They can choose whether or not to accept it. Based on a similar ride-hailing service in the same market, you have some data on which ride requests were accepted and which were not. (The PAY column is what drivers were offered and the ACCEPTED column reflects whether any driver accepted the ride request.)

The demand pool (“riders”) can be acquired at a cost of $30 per rider at any time during the 12 months. There are 10,000 riders in Toledo, but you can’t acquire more than 1,000 in a given month. You start with 0 riders. “Acquisition” means that the rider has downloaded the app and may request rides. Requested rides may or may not be accepted by a driver. In the first month that riders are active, they request rides based on a Poisson distribution where lambda = 1. For each subsequent month, riders request rides based on a Poisson distribution where lambda is the number of rides that they found a match for in the previous month. (As an example, a rider that requests 3 rides in month 1 and finds 2 matches has a lambda of 2 going into month 2.) If a rider finds no matches in a month (which may happen either because they request no rides in the first place based on the Poisson distribution or because they request rides and find no matches), they leave the service and never return.

Proposes a pricing strategy to maximize the profit of the business over the 12 months.

## Solution

The goal is to optimize the offered prices and the numbers of acquisitions for each month to maximize the profit.

We first optimize the offered prices to maximize the total profit for *a single rider* from their time of acquisition to month 12, then use that to model the number of acquisitions per month.

**Intuitions**:
1. The sooner a rider is acquired, the more profit this rider will bring to the program.
2. To retain riders, the offered prices (to drivers) in the earlier months should be higher than the later months.
3. Within a month, the offered prices should increase everytime the same rider make another request until reach a maximum set price. This will increases lambda of the next month and might increase profit in a long run*.

**Assumptions**:
1. Riders' choices of using the service are independent from one another.
2. Within a month, the chance of requesting service is independent from the result of the previous request.
3. In the same month and for the same rider, the same price is offered to driver regardless of number of rides*.
4. Riders in the same cohort (acquired in the same month) go through the same pricing journey. In other words, for these riders, in a given month the offered price is the same.

**Facts and Observations**: Consider a newly acquired rider
* In the first month, they will request *averagely* 1 ride. The chance of this request being accepted is, for example, 79% if $30/ride is offered to the drivers, 50% if $25/ride.
* If we choose to offer drivers $25/ride *for all rides* in the first month, this rider gets 0.5 accepted ride on average. Thus, in month 2, the rider will only request 0.5 ride on average. In the subsequent months, this number will keep reducing and only remain the same when the rate of acceptance is 100% (the offered price > $40)
* If we choose to offer drivers $25/ride *for all rides* in all months, the expected profit of the first month will be $2.5. This value will decrease in the subsequent months. Thus, even if the rider is acquired in month 1, after 12 months, the expected profit of this rider will be < $30, which is the cost of acquisition.

**(*)** To make a simplier model, assumption 3 is created against intuition 3. I am working on an improved model that incorporates different prices when a rider makes 2 or more requests per month.

## Technical details

The rate of acceptance of requested ride given price $x$ of month $i$ following a logistic model:
$$p(x_i) = \frac{1}{1+e^{-(ax_i+b)}}$$

In month $i$, the probability that a rider requests and is accepted for $m$ ride(s) is:
$$P(m, \lambda_i, p_i) = \sum_{j=m}^{\infty} {\frac{j!}{(j-m)!m!}} p_i^m (1-p_i)^{j-m} \frac{\lambda_i^j e^{-\lambda_i}}{j!} = \frac{(\lambda_i p_i)^m e^{-\lambda_i p_i}}{m!}$$
with $m$ is the number of accepted request, $\lambda_i$ is the expected number of requests, and $p_i$ is the rate of acceptance of a requested ride at a given price in month $i$. The final expression is a Poisson distribution with $mean = \lambda p$

The expected revenue from one rider in month i is:
$$R_i = (\lambda_i p_i) * (30-x_i)$$

The lambda for next month is the expected number of accepted ride this month:
$$\lambda_{i+1} = \lambda_{i} p_i$$

Finally, depending on the time of acquisition, let $n$ is the number of months from the month of acquisition to month 12, $n < 12$. The total revenue is:
$$R = \sum_{i=0}^{n} {R_i}$$

### Gradient Descent

$$\frac{\partial R}{\partial x_i} = \sum_{j=i}^{n} {\frac{\partial R_j}{x_i}}$$

With:
$$\frac{\partial R_i}{\partial x_i} = -\lambda_i  p_i^2 (1+e^{-(ax_i+b)} - a(30-x_i)e^{-(ax_i+b)})$$
$$\frac{\partial R_j}{\partial x_i} = \frac{\partial R_j}{\partial\lambda_j} \frac{\partial\lambda_j}{\partial\lambda_{j-1}}...\frac{\partial\lambda_{i+2}}{\partial\lambda_{i+1}} \frac{\partial\lambda_{i+1}}{\partial x_{i}} \text{ with } j>i$$
$$\frac{\partial R_i}{\partial\lambda_i} = p_i (30-x_i)$$
$$\frac{\partial\lambda_{i+1}}{\partial\lambda_{i}} = p_i$$
$$\frac{\partial\lambda_{i+1}}{\partial x_{i}} = \lambda_i * \frac{\partial p_i}{\partial x_i}$$
$$\frac{\partial p_i}{\partial x_i} = a*e^{-(ax+b)}*p_i^2$$

## Result 
* Maximum profit per rider decreases the later they are acquired in the program. In other words, riders acquired in the first month yield highest maximum profit per rider.
* Excluding the cost of acquisition, the highest maximum profit per rider is $5.2. With the $30 cost of acquisition, the program is unprofitable.
