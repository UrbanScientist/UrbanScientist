---
layout: post
title:  "A Mathematical Understanding of the Powerball Lottery"
date:   2019-12-27 08:41:00
categories: Mathematics

---

The popular Powerball Lottery offered in the United States has in the past years, evolved to accommodate larger payouts due to several iterative adjustments in the game's structure. The most recent adjustment was set in place for the drawing which occurred on October 7th, 2015. This probabilistic alteration has opened the opportunity for players to potentially win payouts in excess of $1 billion. This post is the first in a series and seeks to understand how to calculate and comprehend the underlying probabilities or odds associated with the Powerball Lottery game, and to statistically and mathematically explore of the historical outcomes arising from each drawing from the game. <!--more--> If you prefer to computationally walk through the blog post, I will place code snippets within this article for you to run on your own {% sidenote 'One' 'I will be using Python 3.7 for this project. Access the associated [Jupyter Notebook](https://github.com/UrbanScientist/1_Mathematical_Understanding_of_Powerball) for the project here.' %}


## Brief History of the Powerball Lottery Game

Formed in a collaborative effort between the Multi-State Lottery Association (a nonprofit organization), and an agreement with the other US lotteries, the Powerball came into fruition. The Powerball lottery is a Multi-State lottery game offered for to players across 44 States in the US. With the efforts of the Multi-State Lottery Association, the first Powerball drawing was held in 1992. The Powerball became the first lottery game to use two different pools of balls, from which the winning numbers were selected from. This was especially important for the Multi-State Lottery Association, because this two-pool system offered more variation in the numbers for players to select from, implicitly leading to high jackpot odds.

## The Game Structure

For those who are not familiar with how to play the Powerball lottery, the game structure is very simple. There is one player who chooses numbers from two different pools. One can think of a pool as consisting of a range from 1 to *N*, where the range implies that for every integer *x* within it is greater than or equal to 1, and less than or equal to *N*.

 {% math %} Pool = {[1,N] \implies \{ x \in \mathbb{Z} :  1  \leq  x  \leq  N\}}{% endmath %}

The players can select their matrix of numbers by using the "Quick Pick" option, which allows a machine to choose random values for their ticket, or the players can create their own selection matrix on a card which is ran through the system manually.

##  Powerball Ticket Combinations

For the first pool, or the white balls in the Powerball, *N* = 69 making the range of possible selections a range from 1 to 69. The player must select five different numbers from the first pool.

{% math %} First Pool = {[1,69] \implies \{ x \in \mathbb{Z} :  1  \leq  x  \leq  69\}}{% endmath %}

These five numbers, regardless of their order, must match the five numbers drawn from the *FirstPool* by the lottery in order for a chance to win. This type of combination is considered a *combination without replacement*. This idea can be conceptualized in that when the first number is drawn it has a 1 in 69 possibility of matching a ball drawn in the lottery. With the second number there are now only 68 numbers left to draw from, because the numbers are chosen without replacement. Thus, there is now only a 1 in 68 possibility of selecting the matching number. Continuing this logic forward across all the five numbers necessary we arrive at, (1  in  69 x 68 x 67 x 66 x 65) possibilities.

This same concept can be expressed using enumerative combinatorics. Combinatorics broadly is a branch of mathematics whose primary concern is the utilization of certain methodological processes as a means to count, and to extrapolate results from a finite set. When narrowing in on enumerative combinatorics we use the twelvefold way, a framework for counting permutations{% sidenote 'Two' '*Permutation*: An ordinal or sequential arrangement of the components of a set.' %}, combinations {% sidenote 'Three' '*Combination*: The arrangement of a set without reference to the order of the components.' %} and partitions. We will use this use this to define the range of possible combinations in the Powerball lottery.

We can think of the range of possible numbers to be drawn as an immutable set *S* of integers whose bounds are defined in the aforementioned *FirstPool* definition. A k-combination is a finite portion of set *S* with *k* number of distinct elements. If the range of the set is known number *n* is the inclusive maximum element. For the Powerball's *FirstPool* we can define *n* as equal to 69, and *k* as equal to 5. Acknowledging that factorials are computationally heavy, we will use them for simple explanation's sake. Using previously defined *n*, and *k* within equation below we will arrive at the total number of combinations including permutations within the Powerball lottery.

{% math %} \frac{n!}{(n-k)!} {% endmath %}

The computational expression of the binomial coefficient calculation expresses the optionality to include or exclude permutations.

{% highlight python %}
from scipy.special import factorial

def binomial_coefficient(n,k,perm=False):
    '''Calculates the Binomial Coefficient (n choose k)

    Keyword Arguments:
    n = denotes the maximum value in a range from (1,n)
    k = denotes the number of values selected from the n range
    perm = denotes whether permutations are included
    '''
    if perm is True:
        binomial_coefficient = round((factorial(n)) / factorial(n-k))
    else:
        binomial_coefficient = round((factorial(n)) / (factorial(k) * factorial(n-k)))
    return binomial_coefficient
{% endhighlight %}

Using this function we can calculate that number of possible combinations and permutations in the *FirstPool* of the Powerball Lottery.

{% highlight python %} binomial_coefficient(69,5,perm=True) {% endhighlight %}

This leads to **1,348,621,560 combinations and permutations.** Restating once more in order to win the Powerball the order of your selection of numbers is irrelevant, and thus we can eliminate the permutations. We eliminate the permutations by multiplying the denominator by *k!*. {% sidenote 'Five' 'This is accomplished by default with the perm parameter of the binomial_coefficient function.' %}  We do this because given any set of *k* numbers there are *k!* number of ways that *k* could have been drawn. Thus, when we divide the 1,348,621,560 by *k!* we eliminate the effect of the permutations. {% sidenote 'Four' 'This factorial expression of the k-combinatorial calculation in *Eq.2* is also equivalent to the binomial coefficient. Throughout the remainder of this article we will syntactically use the binomial coefficient notation ' %}

{% math %}  \frac{n!}{k!(n-k)!} = \binom{n}{k}  {% endmath %}

Using this function we can calculate that number of possible combinations in the *FirstPool* of the Powerball Lottery.

{% highlight python %} binomial_coefficient(69,5) {% endhighlight %}

This leads to **11,238,513 combinations** in the Powerball lottery's first five numbers. The binomial coefficient can also be thought of as the collection of probabilities. When thinking about the Powerball, the probabilities associated with the first number would be 5 out of 69, the next being 4 out of 68, and so on until it renders the equation below.

{% math %}{n \choose k}={69 \choose 5}={69 \over 5}* {68 \over 4} * {67 \over 3} * {66 \over 2} * {65 \over 1}{% endmath %}

When considering the Powerball, simply calculating the number of combinations across the first pool is not enough. There is a second pool where a red ball is drawn from. {% sidenote 'Five' 'The red ball is the **Powerball**'%} The number of balls in the second pool is 26, making *N* = 26 with the range of possible selections being a range from 1 to 26. The player must select just one number from the second pool.

{% math %} Second Pool = {[1,26] \implies \{ x \in \mathbb{Z} :  1  \leq  x  \leq  26\}}{% endmath %}

In order to calculate the number of combinations including the second pool we modify the aforementioned probabilistic equation slightly.

{% math %}{n \choose k}={69 \choose 5}={69 \over 5}* {68 \over 4} * {67 \over 3} * {66 \over 2} * {65 \over 1} * {26 \over 1}{% endmath %}

With this addition of the second pool the number of combinations increases to **292,201,338** possible combinations. We can refer to the span of all of the different combinations as the *number space*. The ratio of the number of the combinations in play for one drawing relative to the number space is denoted as the *coverage* in a lottery.

## The Odds of Winning

Having discussed how to calculate the possible combinations in the Powerball, let us end by discussing the odds of winning. For now let us just consider the five white balls in the *Firstpool*, and we will discuss the inclusion of the Powerball momentarily. We start with the total number of combinations that are possible if all of the five white balls are selected.

{% math %}{\binom{69}{5}} = 11,238,513 different combinations{% endmath %}

After that, we will define an expression that describes the odds of choosing *n* winning numbers out of the 5 possible winning white balls of the Powerball.

{% math %}{5 \choose n}{% endmath %}

 From this expression we can also think about the inverse of the odds of choosing *n* odds. They would imply that there are 5-n chances to choose a loosing number from the 69-k, or 64 losing numbers.

{% math %}{69-k \choose 5-n} = {64 \choose 5-n}{% endmath %}

Taking both the odds of choosing *n* winning numbers multiplied by the odds of choosing *n* loosing numbers, divided by the odds of choosing all of the winning numbers, we arrive at the odds of choosing *n* correct numbers.

{% math %}{5 \choose n}{64 \choose 5-n}\over {69 \choose 5}{% endmath %}

In our example it is used where *N* is the number of balls in a single pool, *K* is the number of balls in a selection from a single pool, and *B* is the number of matching balls on a single draw.{% sidenote 'Six' 'The generalized equation is known as the hypergeometric distribution.' %}

{% math %}{K \choose B}{N-K \choose K-B} \over {N \choose K}{% endmath %}

This equation would work perfectly except for one final caveat. That being that the Powerball Lottery has two pools which balls are selected from. Just as we did before, the rectification of the equation to accommodate two pools is not difficult. Having only one ball selected from the pool, we can take the probability of it getting selected, multiplied across the numerator of the hypergeometric distribution. With this modification it alters the equation to allow for the selection of 1 ball out of pool *P*, along with the other aforementioned variables.

{% math %}{1 \over P}{K \choose B}{N-K \choose K-B} \over {N \choose K}{% endmath %}

To calculate the the odds of selecting correctly from the *FirstPool*, but incorrectly from the *SecondPool*, we again modify the equation.

{% math %}{P - 1 \over P}{K \choose B}{N-K \choose K-B} \over {N \choose K}{% endmath %}

Computationally, we can derive each of the above equations using the powerball_odds function.

{% highlight python %}
def powerball_odds(n,k,pmax=0):
    '''Calcluates the range of odds of drawing X correct balls in a non-powerball or one-powerball style lottery.

    Keyword Arguments:
    n: The total number of balls in the lottery's drawing
    k: The total number of balls that a player can draw out of the n pool
    pmax:(optional): The maximum value if there is a powerball drawn from a seperate pool.        
    '''
    global odds_list
    global odds_list_PB
    odds_list_PB = []
    odds_list = []
    m = n-k
    for i in range(k+1):
        f = k-i
        odds_a = binomial_coefficient(k,i)
        odds_b = binomial_coefficient(m,f)
        odds_c = binomial_coefficient(n,k)
        if pmax == 0:
            odds = 1/((odds_a * odds_b)/odds_c)
            odds_list.append(odds)
        else:
            odds_d = 1/((odds_a * odds_b)/odds_c)
            odds_PB = odds_d * pmax
            odds_wo_PB = odds_d * pmax / (pmax - 1)
            odds_list_PB.append(odds_PB)
            odds_list.append(odds_wo_PB)
    return odds_list, odds_list_PB
{% endhighlight %}

The code snippet above if ran arrives at two different outputs. Which are two lists of the odds for each type of ticket combination. If you are running on the associated Jupyter notebook, follow the cleaning steps and you will arrive at the correct DataFrames that resemble the following tables.

{% marginnote 'table-1-id' '*Table 1*: The output of the odds for the current Powerball Lottery ' %}
<div class="table-wrapper">
<table class="booktabs">
          <thead>
            <tr><th colspan="2" class="cmid">Output 1: N=69 | P=26</th><th class="nocmid"></th></tr>
            <tr><th>Number Correct</th><th>1/Probability</th><th>Prize</th></tr>
          </thead>
          <tbody>
            <tr><td>0+No Powerball</td> <td>1.53</td>           <td>$0.00</td></tr>
            <tr><td>0+Powerball</td>    <td>38.32</td>          <td>$4.00</td></tr>
            <tr><td>1+No Powerball</td> <td>3.68</td>           <td>$0.00</td></tr>
            <tr><td>1+Powerball</td>    <td>91.98</td>          <td>$4.00</td></tr>
            <tr><td>2+No Powerball</td> <td>28.05</td>          <td>$7.00</td></tr>
            <tr><td>2+Powerball</td>    <td>701.33</td>         <td>$0.00</td></tr>
            <tr><td>3+No Powerball</td> <td>579.76</td>         <td>$7.00</td></tr>
            <tr><td>3+Powerball</td>    <td>14,494.11</td>      <td>$100.00</td></tr>
            <tr><td>4+No Powerball</td> <td>36,525.17</td>      <td>$100.00</td></tr>
            <tr><td>4+Powerball</td>    <td>913,129.18</td>     <td>$50,000.00</td></tr>
            <tr><td>5+No Powerball</td> <td>11,688,053.52</td>  <td>$1,000,000</td></tr>
            <tr><td>5+Powerball</td>    <td>292,201,338.00</td> <td>Jackpot</td></tr>
          </tbody>
</table>
</div>

Prior to October 7th, 2015, the Powerball lottery had the *FirstPool* ranging only to 59, and had the *SecondPool* ranging to 35. The output will allow you to visualize how the change in pool sizes changed the odds of the game and has helped to produce several enormous Powerball payouts!

{% marginnote 'table-1-id' '*Table 2*: The output of the odds for the system that was in place for the Powerball Lottery from January 15, 2012 – October 3, 2015' %}
<div class="table-wrapper">
<table class="booktabs">
          <thead>
            <tr><th colspan="2" class="cmid">Output 1: N=59 | P=35</th><th class="nocmid"></th></tr>
            <tr><th>Number Correct</th><th>1/Probability</th><th>Prize</th></tr>
          </thead>
          <tbody>
            <tr><td>0+No Powerball</td> <td>1.63</td>           <td>$0.00</td></tr>
            <tr><td>0+Powerball</td>    <td>55.41</td>          <td>$4.00</td></tr>
            <tr><td>1+No Powerball</td> <td>3.26</td>           <td>$0.00</td></tr>
            <tr><td>1+Powerball</td>    <td>110.81</td>         <td>$4.00</td></tr>
            <tr><td>2+No Powerball</td> <td>20.78</td>          <td>$0.00</td></tr>
            <tr><td>2+Powerball</td>    <td>706.43</td>         <td>$7.00</td></tr>
            <tr><td>3+No Powerball</td> <td>360.14</td>         <td>$7.00</td></tr>
            <tr><td>3+Powerball</td>    <td>12,244.83</td>      <td>$100.00</td></tr>
            <tr><td>4+No Powerball</td> <td>19,087.53</td>      <td>$100.00</td></tr>
            <tr><td>4+Powerball</td>    <td>648,975.96</td>     <td>$10,000.00</td></tr>
            <tr><td>5+No Powerball</td> <td>5,153,632.65</td>   <td>$1,000,000</td></tr>
            <tr><td>5+Powerball</td>    <td>175,223,510.00</td> <td>Jackpot</td></tr>
          </tbody>
</table>
</div>

## A Quick Comparison between the to tables

When running through the associated Jupyter notebook we see that the Powerball's change in pool size accomplished two things. It first improved the odds of having a winning ticket with the Powerball for tickets with 3 correct numbers or less.Secondly it decreased the odds of having a winning ticket with the Powerball for tickets with 4 or more correct numbers. For tickets without the correct Powerball the odds of winning either stayed the same or increased. This change has allowed the Powerball to increase the coverage of the lottery thus implicitly causing higher jackpot payouts.

## Conclusion

This article covered how to use enumerative combinatorics to calculate the different number of combinations for each of the two pools in the Powerball Lottery, and how to calculate the different odds of winning for each type of ticket. Understanding the mathematics lays the foundation for the next article in Powerball series. Within that next article we will learn how to build a web scraper that gathers historical Powerball lottery data.


### About the author

My name is Jeremy A. Seibert. I built the **Urban Scientist** as a site to host my explorations in the intersection between Economics and Data Science. Most of the posts that I write will have an associated Jupyter notebook accompanying them feel free to check those out at the [Urban Scientist Github ](https://github.com/UrbanScientist). Check out the about page for more info on my background.
