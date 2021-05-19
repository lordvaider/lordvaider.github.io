# Deliveroo Data Analysis II

_This is Part II of a 3 part series. Click here for [Part I]_

In this section, I will do some broad, first-order analysis. No matter how we slice the data (By time periods, cuisine or some other pattern), the first questions that spring to mind are always: 

1. How many orders follow this pattern? 
2. How much money did I spend on such orders? 

Once we answer these in the aggregate, we can do a more in-depth analysis to see how these quantities trend over time, and do a comparative analysis to see how the aggregate values for different slices stack up against each other.

* TOC
{:toc}

# Annual Consumption Trends

For starters, I simply looked at the annual data.

<div width="40%">
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe" style="width: 40%;">
  <thead>
    <tr style="text-align: right;">
      <th>Year</th>
      <th>No_Orders</th>
      <th>No_Items</th>
      <th>Tot_Value</th>
      <th>Avg Order Value</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2018</th>
      <td>88.0</td>
      <td>182.0</td>
      <td>1302.60</td>
      <td>14.80</td>
    </tr>
    <tr>
      <th>2019</th>
      <td>162.0</td>
      <td>406.0</td>
      <td>2681.01</td>
      <td>16.55</td>
    </tr>
    <tr>
      <th>2020</th>
      <td>66.0</td>
      <td>150.0</td>
      <td>1228.76</td>
      <td>18.62</td>
    </tr>
    <tr>
      <th>Total</th>
      <td>316.0</td>
      <td>738.0</td>
      <td>5212.37</td>
      <td>16.49</td>
    </tr>
  </tbody>
</table>
</div>


Broadly speaking, not much change in consumption frequency from 2018 to 2019 (Given that 2018 was half a year worth of data). On average, I ordered in 3 times a week.

Consumption dropped in 2020, for 3 main reasons:
1. Started cooking more at home.
2. Ate out at restaurants/friend's houses more than previous years.
3. Ordered only once during the first 3 months of the Covid19 pandemic.

Next, I decided to plot a graph of my consumption over the course of each year.

# Sidebar: Why are Graphs so painful? 

Having found the total annual consumption, I wanted to plot out the consumption trends to see how they vary over time. This turned out to be more complicated than one would expect, for a variety of reasons. Since this is supposed to be a report of my journey, I thought I'd spend some time fleshing out these complications instead of jumping straight to the finsihed product.

## Subjective Problems
Part of the reason I found plotting graphs difficult was because **I** was plotting them, and I happen to have quite a few mental and emotional hangups. The ones that are relevant here are:

### My Ingratitude and Immaturity
I have incredibly powerful magical abilities that I take for granted. To be fair, this is a shortcoming I share with most of humanity - We take our visualization and graphical processing abilities for granted, and hence underestimate how hard it is to convey visual information to non-visual entities. As an illustrative example look at the scene on your desk. Imagine having to answer a series of simple questions about this scene - Is the lamp to the right of the screen or the left? What is the color of the pen lying closest to the power outlet? Is the stack of papers thicker than the notebook? You'd most likely get a perfect score. Now imagine having to describe the scene to a friend over the phone in enough detail that they can get a perfect score on a similar quiz. Sounds daunting, doesn't it? (By the way, this is a rigorous proof of the folklore theorem: "A picture is worth a thousand words").

Something similar happens when we try to plot graphs on a computer. We are communicating visual information over a text channel, and hence we need to specify "obvious" things that our brain takes for granted - A simple example of is this [Before](https://stackoverflow.com/questions/9603230/how-to-use-matplotlib-tight-layout-with-figure) picture of fig.tight_layout() - A human would _know_ to position the graphs such that the labels don't overlap, but matplotlib needs you to say, "Oh and by the way, please can I have a tight_layout for that fig?!"

Being slammed with unexpected bureaucracy in this way felt unfair and frustrating, and I started throwing tantrums - "Stupid matplotlib developers! Why is something as simple as plotting a graph so complicated?!" The answer, of course, is that plotting a graph isn't that simple - I just felt that way because I have some extremely advanced visual processing machinery sitting between my ears. Eventually I understood that if I wanted graphs, I had to suck it up, be a big boy, and give the machine what it needs.

### My Crippling OCD
I have strong aesthetic preferences about certain things, and find deviations physically painful - For eg. I re-wrote this meta-joke 17 different times, trying to get it just right. 

While plotting these graphs, I had several tiny requirements which I spent a lot of (too much) time on:
 1. When plotting annual consumption graphs for 3 consecutive years, I wanted them to be stacked on top of each other, with the dates aligned. I had to edit the dataset to make sure the dates for each year went from 1 Jan to 31 Dec.
 2. I couldn't choose between plotting number of orders and order value, so I decided to plot both on the same graph. However, when generating the legend, I was only able to generate the legends separately, which meant that either they overlapped and were unreadable, or you had two separate legends in two different corners of the graph, which was super ugly. Finally found a way to hack around this (Thank God for Stackoverflow!) but it should really be a standard option when plotting twinx() charts.
 3. The legend was covering up part of the graph. Set the ylim to be 1.2\*max in order to make room for it.
 4. The dates on the x-axis were rotated. For some reason, these rotated dates really pissed me off, and I wasn't getting the display intervals that I wanted. Is it that hard to label the x-axis with months? Eventually I manged to get the format of the x-axis exactly the way I wanted it, but I'm still not sure how, and don't think I could repeat this feat.

## Objective Problems

There are also some purely technical considerations that make plotting (useful) graphs harder than simply calling a Plot function on a time series.

### Non-Uniformly Distributed Values (Sampling)
My dataset contains points for each order that I placed, and hence is not uniformly sampled. Plotting a line graph on such a dataset could result in some funky looking graphs. When you use a line chart, it will linearly interpolate missing data points, which gives a weird trend line. For eg. it is NOT the case that order activity linearly increased from March to May in the below graph.  

This can be addressed by re-sampling the data into uniformly spaced buckets; For eg. Each day is a bucket and orders for that day feed into that bucket. Days with no orders get assigned zero. This graph correctly shows periods of no activity. 

   
![png](/images/2020-05-17/output_18_0.png)
    


### Discontinuous Jumps (Smoothing)

Resampling the data gives a more accurate representation of order activity, but the resulting graph looks a bit like hedgehog roadkill. A smoother graph would give a better indication of how order activity trends over a period of time.

Signal smoothing is done with low pass filters, which is just a fancy way of saying you need to mathematically transform the series in a way that damps down the  effect of short term fluctuations and pronounces longer term trends (Woah! Looks like the Control Systems course I took in college wasn't a *complete* waste of time!) 
 
One particular way to achieve this is to sum order activity over a lookback window - This also has the advantage of being easy to interpret (Order activity in the past 'n' days). How to pick 'n' in a general scenario is an important question, and hedge funds like WorldQuant hire legions of smart undergrads to <s>try every possible option</s> employ advanced statistical methods to figure it out. In this case however, I just chose a lookback period of one week, because:
   - Food delivery behaviour is roughly periodic over this time period (Tend to order more on the weekends etc). Hence variations on top of this baseline predictability will give us maximal informational payload.
   - Order frequency is typically at least 1 per week; If you pick a look back whose length is less than the average space between orders, there will be no smoothing effect.

__Important Note:__ One last thing to observe from the graph below is that the smoothed graph isn't strictly better, as it sometimes omits juicy details. For eg. On 13 July 2019, I had 5 orders on the same day! 

   
![png](/images/2020-05-17/output_20_0.png)
    


### Sampling + Smoothing?

I'm all about saving effort, and so the natural next thought was: What if, instead of resampling data into daily buckets and then smoothing it out using a weekly window, I just resampled into weekly buckets? As you can see the weekly sampled graph excludes a fair bit of detail, so I decided the short cut wasn't worth it.


    
![png](/images/2020-05-17/output_22_0.png)
    


# Annual Consumption Trends (Graphs)

Putting together all the knowledge from the previous section, I was in a position to plot the graphs tracking the local consumption trends in each year. I wasn't sure about whether to use order value or number of orders as my consumption metric, so I went with both (As I explain below, their interplay also allows us to make some interesting inferences).


![png](/images/2020-05-17/output_24_0.png)
    



    
![png](/images/2020-05-17/output_24_1.png)
    



    
![png](/images/2020-05-17/output_24_2.png)
    


## What do the graphs tell us?
- Going by the scales of the y axes, the weekly consumption dropped in 2020 (In terms of both, the average and the peak value).


- The valleys with zero orders correspond to the times that I was on vacation, or my parents were visiting. The longest period of time without orders was from mid-March 20 to late April 20, which is when Covid19 first went viral (sorry) in the public imgaination.


- The scales of the Value and Orders chart are such that the Orders line (Red) is, in most cases, above the Value line (Green). Hence, the instances where the green line crosses the red line correspond to particularly large orders, when I had guests over (For eg. start Mar 2019, end Apr 2019, mid Nov 2019, start Dec 2020) 
    
    Note that the Value chart has roughly the same height in end April 19 and start May 19, but it's clear that the Value in May came from several orders and was just me pigging out for whatever reason.


# Consumption by Cuisine

The next logical prism to split the data is Cuisine. For starters, what is the distribution of cuisine preference?


<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe" style="width: 50%;">
  <thead>
    <tr style="text-align: right;">
      <th>Cuisine</th>
      <th>OrderNo</th>
      <th>Value</th>
      <th>Value Per Order</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Burger</th>
      <td>89</td>
      <td>1542.18</td>
      <td>17.3</td>
    </tr>
    <tr>
      <th>Indian</th>
      <td>61</td>
      <td>1151.30</td>
      <td>18.9</td>
    </tr>
    <tr>
      <th>Pizza</th>
      <td>28</td>
      <td>646.34</td>
      <td>23.1</td>
    </tr>
    <tr>
      <th>Thai</th>
      <td>33</td>
      <td>586.90</td>
      <td>17.8</td>
    </tr>
    <tr>
      <th>Dessert</th>
      <td>47</td>
      <td>367.05</td>
      <td>7.8</td>
    </tr>
    <tr>
      <th>Chinese</th>
      <td>18</td>
      <td>307.90</td>
      <td>17.1</td>
    </tr>
    <tr>
      <th>Italian</th>
      <td>13</td>
      <td>231.60</td>
      <td>17.8</td>
    </tr>
    <tr>
      <th>Greek</th>
      <td>14</td>
      <td>199.40</td>
      <td>14.2</td>
    </tr>
    <tr>
      <th>Lebanese</th>
      <td>13</td>
      <td>179.70</td>
      <td>13.8</td>
    </tr>
  </tbody>
</table>
</div>


And the same data in pie chart format (As one can see, there is a fair bit of variance in the Value per Order for different cuisines, so Order share seemed like a more democratic metric to compare). Looks like Burgers and Indian food account for about 50% of my consumption!



    
![png](/images/2020-05-17/output_29_0.png)
    


## Cuisine Distribution over Time

I wanted to see how the distribution of various cuisines has varied over time, so plotted the following charts: My cuisine preferences appear to be pretty dynamic! However, this is probably a result of external factors rather than my personality changing from year to year. One such factor is the availability of restaurants on Deliveroo; If Shake Shack delivered to my house in 2018, I'm pretty sure burgers would be the chart-topper that year as well.



    
![png](/images/2020-05-17/output_31_0.png)
    


## Restaurants by Cuisine

One simple question to ask in this regard is, what is the favourite restaurant for each cuisine? As the pie charts above show, ordering behaviour is quite dynamic over time, so it makes sense to look at the favourite restaurant per cuisine per year. (I aggregated over Value in this case, since restaurants with the same cuisine would have prices closer to each other.)




<div width="50%>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe" style="width: 50%;">
  <thead>
    <tr style="text-align: right;">
      <th>Year</th>
      <th>2018</th>
      <th>2019</th>
      <th>2020</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Burger</th>
      <td>Byron</td>
      <td>Shake Shack</td>
      <td>Shake Shack</td>
    </tr>
    <tr>
      <th>Chinese</th>
      <td>Grilled Fusion</td>
      <td>Ping Pong</td>
      <td></td>
    </tr>
    <tr>
      <th>Dessert</th>
      <td>Cookies &amp; Cream</td>
      <td>Cookies &amp; Cream</td>
      <td>Craving Dessert</td>
    </tr>
    <tr>
      <th>Greek</th>
      <td>The Athenian</td>
      <td>The Athenian</td>
      <td>The Athenian</td>
    </tr>
    <tr>
      <th>Indian</th>
      <td>Namma by Kricket</td>
      <td>Motu Indian Kitchen</td>
      <td>Dishoom</td>
    </tr>
    <tr>
      <th>Italian</th>
      <td>La Figa</td>
      <td></td>
      <td>Scarpetta</td>
    </tr>
    <tr>
      <th>Lebanese</th>
      <td>The Chickpea</td>
      <td>Waleema</td>
      <td>Efes</td>
    </tr>
    <tr>
      <th>Pizza</th>
      <td>PizzaExpress</td>
      <td>The Pizza Room</td>
      <td>The Pizza Room</td>
    </tr>
    <tr>
      <th>Thai</th>
      <td>Rusty Bike</td>
      <td>Rusty Bike</td>
      <td>Busaba</td>
    </tr>
  </tbody>
</table>
</div>


Also included a fancy Tableau graph of this data, since just showing the max value restaurant hides close runners up (As is the case with Byron and Shake Shack in 2019).

![png](/images/2020-05-17/cusine_year_rest.PNG)

# Consumption by Restaurant

Next, I broke down the data by restaurant. Here again, I looked at the overall total, and then looked at the distribution on a year by year basis.



<div width="50%">
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe" style="width: 50%;">
  <thead>
    <tr style="text-align: right;">
      <th>rName</th>
      <th>OrderNo</th>
      <th>Value</th>
      <th>Avg Value</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Shake Shack</th>
      <td>47</td>
      <td>707.40</td>
      <td>15.1</td>
    </tr>
    <tr>
      <th>Byron</th>
      <td>31</td>
      <td>679.30</td>
      <td>21.9</td>
    </tr>
    <tr>
      <th>Motu Indian Kitchen</th>
      <td>32</td>
      <td>565.75</td>
      <td>17.7</td>
    </tr>
    <tr>
      <th>The Pizza Room</th>
      <td>18</td>
      <td>419.54</td>
      <td>23.3</td>
    </tr>
    <tr>
      <th>Rusty Bike</th>
      <td>21</td>
      <td>346.10</td>
      <td>16.5</td>
    </tr>
    <tr>
      <th>Busaba</th>
      <td>12</td>
      <td>240.80</td>
      <td>20.1</td>
    </tr>
    <tr>
      <th>Ping Pong</th>
      <td>12</td>
      <td>238.25</td>
      <td>19.9</td>
    </tr>
    <tr>
      <th>PizzaExpress</th>
      <td>9</td>
      <td>212.25</td>
      <td>23.6</td>
    </tr>
    <tr>
      <th>The Athenian</th>
      <td>14</td>
      <td>199.40</td>
      <td>14.2</td>
    </tr>
    <tr>
      <th>Cookies &amp; Cream</th>
      <td>27</td>
      <td>192.15</td>
      <td>7.1</td>
    </tr>
    <tr>
      <th>Manjal</th>
      <td>7</td>
      <td>177.95</td>
      <td>25.4</td>
    </tr>
    <tr>
      <th>Dishoom</th>
      <td>6</td>
      <td>166.20</td>
      <td>27.7</td>
    </tr>
  </tbody>
</table>
</div>


## Restaurant Distribution over time

The pie charts below the order share of restaurants per year. I only included restaurants with > 5% order share to keep things readable.



![png](/images/2020-05-17/output_39_0.png)
    


I don't know if someone who doesn't know me could figure out much about me from the charts above, but they make a lot of sense to me given what I know about myself.

__2018:__ I first moved to London, and had a hankering for Indian food (Namma by Kricket) and ordered a lot from Pizza Express (The power of brand recognition). Namma by Kricket shut shop within a couple of months (The power of a really bad name) and Rusty Bike became my go-to for simple, not-too-unhealthy food. Once I discovered Pizza Room, I entirely switched over to them for all my pizza needs. Cookies and Cream was the generic cake shop closest to my house.

__2019:__ Early 2019 was a wild time. Two of my favourite restaurants, Shake Shack and Ping Pong started delivery to my house. Unfortunately, I moved houses and Ping Pong no longer delivered to my new house, which explains the 8.9% above. Also, Byron launched a veggie burger (Limited edition though), which meant I ordered from them a lot more. Motu was another one of the new entrants at this time - They had a Box for 1, which was strictly average in quality but sated my desire for Indian food. Dzrt was the generic cake shop closest to my new house.

__2020:__ Two cataclysmic events occurred in late 2019; A Chinese dude ate a bat sandwich and I got a new flatmate. My flatmate didn't enjoy burgers as much as I did, which meant a drastic reduction in Byron + Shake Shack. Also, the pandemic seemed to affect Byron particularly badly (If Lord Byron is reading this, check out Section 3 for some ideas to increase business). The preponderance of Busaba can be attributed to my flatmate, while Capeesh was all mine. We both got behind the Athenian (Halloumi Souvlaki), Manjal (Uthappa aka Savory South Indian rice pancakes) and Scarpetta (Pasta for her, grilled chicken+veggies for me). Due to the lockdown, Dishoom finally entered the food delivery game in the later part of the year. Another pandemic baby was Chowpatty, an upstart, home-run 'restaurant' that delivered Bombay street food (Complete with raw mango garnish).

# Look Ma! Bar Charts!

Honestly, this section is just me flexing my newly developed plotting muscles...

    
![png](/images/2020-05-17/output_42_0.png)
    

Unsurprisingly, most of the food is ordered on the weekends. But why stop here? Let's take a look at:


![png](/images/2020-05-17/output_44_0.png)
    


Another anti-surprise, most of the food ordered during lunch and dinner, with more dinner orders than lunch orders. And because we live in an age of cheap compute, I decided to also plot...


    
![png](/images/2020-05-17/output_46_0.png)
    


OHMYGODOHMYGODOHMYGOD!!! No orders during the [23rd](https://en.wikipedia.org/wiki/The_Number_23) minute!!! Clearly I'm [living in a movie](https://en.wikipedia.org/wiki/The_Truman_Show)...

# Cost Analysis

As calculated before, my total spend on Deliveroo so far is __£5212.37__, which works out to __£87 per month__ on average. 

## Distribution of Order Values

The average price of an order is £16.49 and the median is £15.77. I've plotted the distribution below, and it's pretty obviously multimodal - For eg. The large concentration of orders at about £8 corresponds to the dessert orders. One can also see the larger orders when I ordered for a group of people (Seems like there were at least 7/8 such occasions).


    
![png](/images/2020-05-17/output_49_0.png)
    


## Is my Deliveroo Plus Account worth it?

This calculation is very hard to do super accurately, since the rules of the game keep changing. For eg. in Jan 20, Deliveroo introduced a £10 minimum order value to avail free delivery. The delivery fee structure is also pretty complicated, though Deliveroo states that on average, it is about £2.5. I have no idea how that number has changed over time though - Most of my email receipts before subscribing to Plus state the delivery fee is £0, which makes me wonder why I subscribed in the first place. To cut a long story short I will proceed with the following assumptions - Delivery fee would have been £2.5 without Plus, and the £10 threshold applies to all orders.

I started my Plus subscription in October 2019. At the time, it cost £7.99 per month (Increased to 11.49 per month in December 20). Over 15 months, this is a total spend of £123.35.

Over the same period of time, I had 80 orders that cost more than £10. This translates to savings on delivery fees of £200, and __net savings of £66.65__ (So close!) or a princely sum of __£4.44 per month__. At the present cost of Plus, the net savings would be __less than £2 per month__.

It would be nice if Deliveroo themselves could do this calculation for you. Obviously, they wouldn't want to show you that info if it turns out your Plus membership is actually a net negative for you, and I don't think it would be acceptable for them to only show this statistic to people who benefitted from Plus. 

Hence, the only way such a feature could work is if they showed people who don't have Plus how much they would have benefitted with Plus given their order history - "If you had signed up for Plus in Oct 2019, and used the savings to purchase long-dated Gamestop options, today you'd have a 1000 Dogecoins!". Of course, this is assuming that Deliveroo is actually incentivized to convert such customers to Plus; I've no idea how the economics on that works.

## Most Expensive Restaurant

This should be an easy one, right? The most expensive restaurant is the one that costs the highest price per unit of food, which seems straightforward enough. Unfortunately, the hard part of using that formula is defining a 'unit of food'. Do we define an order to constitute 1 unit? This doesn't seem right, as we will see in the analysis of individual restaurants, order sizes can be quite variable even for orders from the same restaurant (Due to guests for example). 

How about defining a unit of food to be one restaurant item? This is even more problematic, because items can fall into various categories - Mains, Sides, Pizzas (Yes, pizza is a separate category!), Drinks, Desserts etc. and are hence even less uniform than orders. There might be a path here if we manage to categorize the items, and take some sorted of weighted sum across categories (1 Side = 0.5 Units, 1 Main = 1 Unit, 1 Pizza = 1.5 Units etc.) but the categorization is a challenge in it's own right (That I explore in the restaurant level analysis). 

Another approach would be to go full Physics and define food units in calories, but caloric information doesn't exist for most of these items - Hit me up if you can think of a not-too-hard, non-manual way to come up with approximate calorie counts for the items! Also,  caloric count isn't proportional to satiety - The calorie approach would be biased against desserts, but that is just an argument for why desserts should be their own category.

In the absence of clear answers, I decided to just calculate the 2 simplest metrics (Average Order Value and Average Item Value), and see which one made more sense.


<div width="20%">
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>

<table border="1" class="dataframe" style="width: 20%;">
  <thead>
    <tr style="text-align: right;">
      <th>rName</th>
      <th>Average Item Value</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Motu Indian Kitchen</td>
      <td>11.55</td>
    </tr>
    <tr>
      <td>The Pizza Room</td>
      <td>10.76</td>
    </tr>
    <tr>
      <td>PizzaExpress</td>
      <td>10.61</td>
    </tr>
    <tr>
      <td>Busaba</td>
      <td>8.30</td>
    </tr>
    <tr>
      <td>Cookies &amp; Cream</td>
      <td>7.12</td>
    </tr>
    <tr>
      <td>Manjal</td>
      <td>7.12</td>
    </tr>
    <tr>
      <td>The Athenian</td>
      <td>6.88</td>
    </tr>
    <tr>
      <td>Dishoom</td>
      <td>6.65</td>
    </tr>
    <tr>
      <td>Rusty Bike</td>
      <td>6.41</td>
    </tr>
    <tr>
      <td>Byron</td>
      <td>6.01</td>
    </tr>
  </tbody>
</table>
</div>

Off the bat, we can see that Item Value as a metric is giving nonsensical results - The top 3 items have Pizza restaurants (As predicted) and Motu Indian Kitchen, which has the massive "Box for 1" as a single item, but is hardly an expensive/high class restaurant. The other entries on the list are equally nonsensical (The Athenian? Rusty Bike?? Cookies & Cream???)



<div width="20%">
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>

<table border="1" class="dataframe" style="width: 20%;">
  <thead>
    <tr style="text-align: right;">
      <th>rName</th>
      <th>Average Order Value</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Dishoom</td>
      <td>27.70</td>
    </tr>
    <tr>
      <td>Manjal</td>
      <td>25.42</td>
    </tr>
    <tr>
      <td>PizzaExpress</td>
      <td>23.58</td>
    </tr>
    <tr>
      <td>The Pizza Room</td>
      <td>23.31</td>
    </tr>
    <tr>
      <td>Byron</td>
      <td>21.91</td>
    </tr>
    <tr>
      <td>Busaba</td>
      <td>20.07</td>
    </tr>
    <tr>
      <td>Ping Pong</td>
      <td>19.85</td>
    </tr>
    <tr>
      <td>Motu Indian Kitchen</td>
      <td>17.68</td>
    </tr>
    <tr>
      <td>Chowpatty</td>
      <td>17.11</td>
    </tr>
    <tr>
      <td>Rusty Bike</td>
      <td>16.48</td>
    </tr>
  </tbody>
</table>
</div>

This seems more in line with the truth, but again there is a very evident bias - All of Dishoom, Manjal and PizzaExpress have instances of large (> £60) orders, which skew their average order value upwards. 

Another point in favour of the order metric - This list features the fancier places like Ping Pong, Busaba, and Byron higher up than the first list.

In order to see the restaurant-wise anallysis analysis, check out [Section III]!