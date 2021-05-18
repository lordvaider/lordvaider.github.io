# Deliveroo Data Analysis

* TOC
{:toc}

In this notebook, my goal is to analyse my Deliveroo order data starting May 2018 and see what I can learn about myself from it.

This writeup is divided into 4 sections:
 1. __Fetching the raw data:__ Provides details on the steps taken to parse email order receipts and obtain the data in a structured format.
 2. __Top level analysis:__ Simple things like spend per year, consumption patterns over time, distribution by cuisine and how it changed over time. Also includes a behind the scenes look at why these patterns changed in the way they did. Includes various different strategies I used to plot graphs and their relative merits.
 3. __Restaurant wise analysis:__ For most popular restaurants, calculate some basic stats for each, such as total spend, number of orders, average order size etc. Plot popularity over time. Get a list of most popular items. Also includes reviews and other personal tidbits.
 4. __Future Work:__ What else can we learn from this data that isn't food related? We can know when we weren't at home for eg. We can also know when we had guests over. Can we know about other events/phase shifts based on some changes in order activity?


# Step 0 - Build Mad Skillz

Despite having an advanced degree in Computer Science and working as a sort-of-software engineer for several years, I knew embarrasingly little about the basic machinery required to work on this project. Hence I spent a fair bit of time just learning some super simple stuff:

1. Python: I just went through the basic tutorial on Kaggle, so list comprehension is the most advanced thing I know, but that was already enough to do a lot. 

2. Pandas: For those that don't know, Pandas is a Python library used for dealing with tabular data sets (Dataframes) - Pretty much bread and butter for any datadude. Again, going through the Kaggle tutorial was sufficient to get started.

3. Jupyter Notebooks: I guess the only thing I had to learn was the fact that they exist! I really love it when tools are powerful AND easy to use - I felt like I had been looking for Jupyter notebooks my entire life!

4. Other tools: Played around with a couple of different code editors, picked up the basics of the command line and git and went through some of the lectures [here](https://missing.csail.mit.edu/).  

5. Touch Typing: Imagine having to use a pencil taped to a brick every time you want to write something. That's what using a keyboard felt like to me. I decided to make the interface between my brain and the computer as seamless as possible, and learnt to touch type. Best investment of my time ever!

![png](/images/2020-05-17/engelbart.PNG)

# Getting the data
The first step of any data project is to get the data into a usable format (Most often a table), clean it (Get rid off meaningless/null values) and enrich with derived fields that will be utilized in the analysis.

## Parsing Email Receipts to .csv
As a first step, I wrote some code to parse the email receipts that I get from Deliveroo each time I place an order and extract the data into a nice .csv file. This turned out to be harder than I originally thought it would be. Some challenges I faced along the way:

 1. __Deciding which format to parse:__ Each email contains a text version and an HTML version. Initially, parsing the HTML seemed like the more correct way; There was even a neat pandas function that converted from HTML to a dataframe! 
     
     However the HTML to dataframe function was super brittle, and when it broke, I didn't exactly know why. Rather than dig through the error messages and try to make the function work, I just wrote a backup text parser for the cases where the HTML parser failed. The text parser turned out to be much simpler and, as I found later, more accurate as well.
     
     There are libraries such as Beautiful soup whose specific purpose is to scrape data from HTML, but I decided to postpone digging into them till I didn't have a choice.
     

 2. __The format of the email changes over time:__ I was expecting this to be an issue. As a simple example, the top line of the Deliveroo email reads: "{Restaurant Name} has your order!". However, before May 2019, it used to read "{Restaurant Name} has __accepted__ your order!". 
    
    The first solution my brain suggested was to put a branch in my code to check whether the order was received before/after 1 May 2019, and parse the restaurant name accordingly. Of course, this solution was terribly unsatisfactory because:
     - I didn't know exactly when the format had changed, I just knew that it had changed at some point between two of my orders. Hence, there was a chance that if someone else ran this code for their orders, it might parse them incorrectly.
     - More seriously, the format could change again at some point, and then I'd have to add yet another branch in my code, increasing the length and complexity of my codebase.

    I shrugged off these concerns and just coded it up anyway, since I wanted to finish the data scraping ASAP and move on to the __ANALYSIS!__. However the code didn't work as expected since before November 2018, the top line used to read "{Restaurant Name} has __received__ your order!" 
    
    There was no way I was putting 3 branches in my code, and so I decided to respect the problem and actually think about it for 5 minutes. At 4 minutes and 20 seconds, I realized I could leverage some regular expression magic to vastly simplify the code. 
    
    For those that don't know, regular expressions are a way to recognize if some text data matches a certain pattern, and split it into sub-patterns. I was using a different regexp in each branch to extract the restaurant name:
    - {Restaurant Name} has your order!
    - {Restaurant Name} has accepted your order!
    - {Restaurant Name} has received your order!
    
    However, as I later realised, I could actually push the branching into the regexp itself, and use just one: "{Restaurant Name} has (\|accepted\|received) your order!" This change avoided blowing up the size of the code and also exorcised the date-based check.       

 3. __The price convention seemed to change randomly:__ This was a subtle issue, and I actually noticed it when I was deep in the __ANALYSIS!__ stage. I was plotting the distribution of order sizes, and found that I had spent £150 on a single order at Dishoom, which I did not remember. 
 

    Digging further, I found that while most of the receipts contained the unit price of each item, some of them (Notably, the ones for Dishoom) had the total price (Unit price x Item Quantity). I was working under the assumption that they were all unit prices, and hence the totals for Dishoom were being calculated as much higher than they actually were.
    
    
    In order to figure out which receipt followed which convention, I started parsing the bottomline numbers (Sub-Total, Delivery Fee, Taxes, Total) in the receipt. I then did a basic checksum against the implied total under both price conventions, to see which receipt followed which convention.
     
     
     Eventually though, this turned out to be wasted effort. The two different price conventions were an artefact introduced by some extra display logic in the HTML code, and the text part of the email had the right values all along. However, it's still good to have the bottom line numbers in order to analyse things like how delivery fees etc changed over time/How much I pay in extra charges etc. so this was't a _total_ waste.

I have shared the end result of these efforts in git repo here, in case any of you are interested in fetching your own Deliveroo data.

Eventually, I was able to get my data extraction working and got the raw data corresponding to (almost) all orders in a neat .csv file. Just scrolling through this file, I got a feeling of power. <s>Finally it was __ANALYSIS!__ time!</s> Finally, it was time to clean my data and get it into the right format!

## Data Formatting + Cleaning

In order to do useful ANALYSIS! on the data, it first needed some cleaning and formatting: 
1. __Datatype Conversions:__ Need to convert the values of the "Date" column into datetimes.


2. __Add Inferred Columns__: Enrich the dataset with some more columns that will be required in the course of the analysis such as Value, and OrderNo (All items ordered at the same time share an order no.)


3. __Remove outliers:__ Remove rows with price = 0. Typically, such rows correspond to freebies such as ketchup/mustard packets. While it may be interesting to analyse such rows separately, I will exclude them for now.


4. __String Cleaning:__ Some of the restaurant names and item names contained weird characters such as "=E2=80=99" -- This is a consequence of [UTF-8](https://en.wikipedia.org/wiki/UTF-8) characters in the item names. UTF-8 is an encoding scheme by means of which emojis and special symbols are represented using letters and numbers. Examples: 
    - "=C2=AE" is the UTF-8 hex encoding of the subtly threatening (R) that follows a registered trademark. 
    - "=F0=9F=8C=B6" represents the cute chili emojis that serve as a spice warning. 
    - A lot of them are Mandarin characters, representing the original Chinese name of the item.
    
    I don't care about any of these (Least of all the spice warnings!), and only remapped =E2=80=99 to the single apostrophe to make things a little more readable.


5. __Add External Context:__ In order to make the analysis more meaningful, I wanted to add in some contextual data that is not present in the raw Deliveroo data:
    - __Map to Real Name:__ Sometimes, restaurants pop up with slightly different names, because of branding exercises or different branches - I mapped these to a single name (The rName or real name). This is important when trying to answer questions like which restaurant is the most popular one.
    - __Add Cuisine:__ Knowing what kind of cuisine each restaurant serves was important to gain a broad understanding of my tastes and how they evolve over time. It was also important for the purposes of analysis. Example: If you want to calculate the average price of a meal, you may want to exclude dessert orders since these are much smaller on average.

   While it may be possible to automate the creation of this contextual dataset, I just did it manually. There are only 49 distinct restaurants that I ordered from and hence labelling them took less than 5 minutes.


Once the data had been extracted, cleaned, formatted and enriched with external context, I spent 5 minutes gazing at it lovingly before diving into the __ANALYSIS!__


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
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Restaurant</th>
      <th>Item</th>
      <th>Qty</th>
      <th>Price</th>
      <th>rName</th>
      <th>Cuisine</th>
      <th>Value</th>
      <th>OrderNo</th>
      <th>Year</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2018-05-06 20:38:59</td>
      <td>Mother Clucker Editions</td>
      <td>Halloumi Bun</td>
      <td>1</td>
      <td>16.5</td>
      <td>Mother Clucker Editions</td>
      <td>Burger</td>
      <td>16.5</td>
      <td>0</td>
      <td>2018</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2018-05-07 11:42:05</td>
      <td>Namma by Kricket</td>
      <td>Aloo Chaat</td>
      <td>1</td>
      <td>5.5</td>
      <td>Namma by Kricket</td>
      <td>Indian</td>
      <td>5.5</td>
      <td>1</td>
      <td>2018</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2018-05-07 11:42:05</td>
      <td>Namma by Kricket</td>
      <td>Burnt Garlic Tarka Dhal</td>
      <td>1</td>
      <td>3.5</td>
      <td>Namma by Kricket</td>
      <td>Indian</td>
      <td>3.5</td>
      <td>1</td>
      <td>2018</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2018-05-07 11:42:05</td>
      <td>Namma by Kricket</td>
      <td>Matar Pilau</td>
      <td>1</td>
      <td>3.2</td>
      <td>Namma by Kricket</td>
      <td>Indian</td>
      <td>3.2</td>
      <td>1</td>
      <td>2018</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2018-05-07 11:42:05</td>
      <td>Namma by Kricket</td>
      <td>Papad</td>
      <td>1</td>
      <td>1.0</td>
      <td>Namma by Kricket</td>
      <td>Indian</td>
      <td>1.0</td>
      <td>1</td>
      <td>2018</td>
    </tr>
  </tbody>
</table>
</div>



# Top Level Analysis

In this section, I will do some broad, first-order analysis. No matter how we slice the data (By time periods, cuisine or some other pattern), the first questions that spring to mind are always: 

1. How many orders follow this pattern? 
2. How much money did I spend on such orders? 

Once we answer these in the aggregate, we can do a more in-depth analysis to see how these quantities trend over time, and do a comparative analysis to see how the aggregate values for different slices stack up against each other.

## Annual Consumption Trends

For starters, I simply looked at the annual data.



| Year  | No_Orders | No_Items | Tot_Value | Avg Order Value |
|-------|-----------|----------|-----------|-----------------|
| 2018  | 88.0      | 182.0    | 1302.60   | 14.80           |
| 2019  | 162.0     | 406.0    | 2681.01   | 16.55           |
| 2020  | 66.0      | 150.0    | 1228.76   | 18.62           |
| Total | 316.0     | 738.0    | 5212.37   | 16.49           |


Broadly speaking, not much change in consumption frequency from 2018 to 2019 (Given that 2018 was half a year worth of data). On average, I ordered in 3 times a week.

Consumption dropped in 2020, for 3 main reasons:
1. Started cooking more at home.
2. Ate out at restaurants/friend's houses more than previous years.
3. Ordered only once during the first 3 months of the Covid19 pandemic.

Next, I decided to plot a graph of my consumption over the course of each year.

## Sidebar: Why are Graphs so painful? 

Having found the total annual consumption, I wanted to plot out the consumption trends to see how they vary over time. This turned out to be more complicated than one would expect, for a variety of reasons. Since this is supposed to be a report of my journey, I thought I'd spend some time fleshing out these complications instead of jumping straight to the finsihed product.

### Subjective Problems
Part of the reason I found plotting graphs difficult was because **I** was plotting them, and I happen to have quite a few mental and emotional hangups. The ones that are relevant here are:

#### My Ingratitude and Immaturity
I have incredibly powerful magical abilities that I take for granted. To be fair, this is a shortcoming I share with most of humanity - We take our visualization and graphical processing abilities for granted, and hence underestimate how hard it is to convey visual information to non-visual entities. As an illustrative example look at the scene on your desk. Imagine having to answer a series of simple questions about this scene - Is the lamp to the right of the screen or the left? What is the color of the pen lying closest to the power outlet? Is the stack of papers thicker than the notebook? You'd most likely get a perfect score. Now imagine having to describe the scene to a friend over the phone in enough detail that they can get a perfect score on a similar quiz. Sounds daunting, doesn't it? (By the way, this is a rigorous proof of the folklore theorem: "A picture is worth a thousand words").

Something similar happens when we try to plot graphs on a computer. We are communicating visual information over a text channel, and hence we need to specify "obvious" things that our brain takes for granted - A simple example of is this [Before](https://stackoverflow.com/questions/9603230/how-to-use-matplotlib-tight-layout-with-figure) picture of fig.tight_layout() - A human would _know_ to position the graphs such that the labels don't overlap, but matplotlib needs you to say, "Oh and by the way, please can I have a tight_layout for that fig?!"

Being slammed with unexpected bureaucracy in this way felt unfair and frustrating, and I started throwing tantrums - "Stupid matplotlib developers! Why is something as simple as plotting a graph so complicated?!" The answer, of course, is that plotting a graph isn't that simple - I just felt that way because I have some extremely advanced visual processing machinery sitting between my ears. Eventually I understood that if I wanted graphs, I had to suck it up, be a big boy, and give the machine what it needs.

#### My Crippling OCD
I have strong aesthetic preferences about certain things, and find deviations physically painful - For eg. I re-wrote this meta-joke 17 different times, trying to get it just right. 

While plotting these graphs, I had several tiny requirements which I spent a lot of (too much) time on:
 1. When plotting annual consumption graphs for 3 consecutive years, I wanted them to be stacked on top of each other, with the dates aligned. I had to edit the dataset to make sure the dates for each year went from 1 Jan to 31 Dec.
 2. I couldn't choose between plotting number of orders and order value, so I decided to plot both on the same graph. However, when generating the legend, I was only able to generate the legends separately, which meant that either they overlapped and were unreadable, or you had two separate legends in two different corners of the graph, which was super ugly. Finally found a way to hack around this (Thank God for Stackoverflow!) but it should really be a standard option when plotting twinx() charts.
 3. The legend was covering up part of the graph. Set the ylim to be 1.2\*max in order to make room for it.
 4. The dates on the x-axis were rotated. For some reason, these rotated dates really pissed me off, and I wasn't getting the display intervals that I wanted. Is it that hard to label the x-axis with months? Eventually I manged to get the format of the x-axis exactly the way I wanted it, but I'm still not sure how, and don't think I could repeat this feat.

### Objective Problems

There are also some purely technical considerations that make plotting (useful) graphs harder than simply calling a Plot function on a time series.

#### Non-Uniformly Distributed Values (Sampling)
My dataset contains points for each order that I placed, and hence is not uniformly sampled. Plotting a line graph on such a dataset could result in some funky looking graphs. When you use a line chart, it will linearly interpolate missing data points, which gives a weird trend line. For eg. it is NOT the case that order activity linearly increased from March to May in the below graph.  

This can be addressed by re-sampling the data into uniformly spaced buckets; For eg. Each day is a bucket and orders for that day feed into that bucket. Days with no orders get assigned zero. This graph correctly shows periods of no activity. 

   
![png](/images/2020-05-17/output_18_0.png)
    


#### Discontinuous Jumps (Smoothing)

Resampling the data gives a more accurate representation of order activity, but the resulting graph looks a bit like hedgehog roadkill. A smoother graph would give a better indication of how order activity trends over a period of time.

Signal smoothing is done with low pass filters, which is just a fancy way of saying you need to mathematically transform the series in a way that damps down the  effect of short term fluctuations and pronounces longer term trends (Woah! Looks like the Control Systems course I took in college wasn't a *complete* waste of time!) 
 
One particular way to achieve this is to sum order activity over a lookback window - This also has the advantage of being easy to interpret (Order activity in the past 'n' days). How to pick 'n' in a general scenario is an important question, and hedge funds like WorldQuant hire legions of smart undergrads to <s>try every possible option</s> employ advanced statistical methods to figure it out. In this case however, I just chose a lookback period of one week, because:
   - Food delivery behaviour is roughly periodic over this time period (Tend to order more on the weekends etc). Hence variations on top of this baseline predictability will give us maximal informational payload.
   - Order frequency is typically at least 1 per week; If you pick a look back whose length is less than the average space between orders, there will be no smoothing effect.

__Important Note:__ One last thing to observe from the graph below is that the smoothed graph isn't strictly better, as it sometimes omits juicy details. For eg. On 13 July 2019, I had 5 orders on the same day! 

   
![png](/images/2020-05-17/output_20_0.png)
    


#### Sampling + Smoothing?

I'm all about saving effort, and so the natural next thought was: What if, instead of resampling data into daily buckets and then smoothing it out using a weekly window, I just resampled into weekly buckets? As you can see the weekly sampled graph excludes a fair bit of detail, so I decided the short cut wasn't worth it.


    
![png](/images/2020-05-17/output_22_0.png)
    


## Annual Consumption Trends (Graphs)

Putting together all the knowledge from the previous section, I was in a position to plot the graphs tracking the local consumption trends in each year. I wasn't sure about whether to use order value or number of orders as my consumption metric, so I went with both (As I explain below, their interplay also allows us to make some interesting inferences).


![png](/images/2020-05-17/output_24_0.png)
    



    
![png](/images/2020-05-17/output_24_1.png)
    



    
![png](/images/2020-05-17/output_24_2.png)
    


### What do the graphs tell us?
- Going by the scales of the y axes, the weekly consumption dropped in 2020 (In terms of both, the average and the peak value).


- The valleys with zero orders correspond to the times that I was on vacation, or my parents were visiting. The longest period of time without orders was from mid-March 20 to late April 20, which is when Covid19 first went viral (sorry) in the public imgaination.


- The scales of the Value and Orders chart are such that the Orders line (Red) is, in most cases, above the Value line (Green). Hence, the instances where the green line crosses the red line correspond to particularly large orders, when I had guests over (For eg. start Mar 2019, end Apr 2019, mid Nov 2019, start Dec 2020) 
    
    Note that the Value chart has roughly the same height in end April 19 and start May 19, but it's clear that the Value in May came from several orders and was just me pigging out for whatever reason.


## Consumption by Cuisine

The next logical prism to split the data is Cuisine. For starters, what is the distribution of cuisine preference?


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
      <th></th>
      <th>OrderNo</th>
      <th>Value</th>
      <th>Value Per Order</th>
    </tr>
    <tr>
      <th>Cuisine</th>
      <th></th>
      <th></th>
      <th></th>
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
    


### Cuisine Distribution over Time

I wanted to see how the distribution of various cuisines has varied over time, so plotted the following charts: My cuisine preferences appear to be pretty dynamic! However, this is probably a result of external factors rather than my personality changing from year to year. One such factor is the availability of restaurants on Deliveroo; If Shake Shack delivered to my house in 2018, I'm pretty sure burgers would be the chart-topper that year as well.



    
![png](/images/2020-05-17/output_31_0.png)
    


### Restaurants by Cuisine

One simple question to ask in this regard is, what is the favourite restaurant for each cuisine? As the pie charts above show, ordering behaviour is quite dynamic over time, so it makes sense to look at the favourite restaurant per cuisine per year. (I aggregated over Value in this case, since restaurants with the same cuisine would have prices closer to each other.)




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
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>Year</th>
      <th>2018</th>
      <th>2019</th>
      <th>2020</th>
    </tr>
    <tr>
      <th>Cuisine</th>
      <th></th>
      <th></th>
      <th></th>
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

## Consumption by Restaurant

Next, I broke down the data by restaurant. Here again, I looked at the overall total, and then looked at the distribution on a year by year basis.



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
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>OrderNo</th>
      <th>Value</th>
      <th>Avg Value</th>
    </tr>
    <tr>
      <th>rName</th>
      <th></th>
      <th></th>
      <th></th>
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


### Restaurant Distribution over time

The pie charts below the order share of restaurants per year. I only included restaurants with > 5% order share to keep things readable.



![png](/images/2020-05-17/output_39_0.png)
    


I don't know if someone who doesn't know me could figure out much about me from the charts above, but they make a lot of sense to me given what I know about myself.

__2018:__ I first moved to London, and had a hankering for Indian food (Namma by Kricket) and ordered a lot from Pizza Express (The power of brand recognition). Namma by Kricket shut shop within a couple of months (The power of a really bad name) and Rusty Bike became my go-to for simple, not-too-unhealthy food. Once I discovered Pizza Room, I entirely switched over to them for all my pizza needs. Cookies and Cream was the generic cake shop closest to my house.

__2019:__ Early 2019 was a wild time. Two of my favourite restaurants, Shake Shack and Ping Pong started delivery to my house. Unfortunately, I moved houses and Ping Pong no longer delivered to my new house, which explains the 8.9% above. Also, Byron launched a veggie burger (Limited edition though), which meant I ordered from them a lot more. Motu was another one of the new entrants at this time - They had a Box for 1, which was strictly average in quality but sated my desire for Indian food. Dzrt was the generic cake shop closest to my new house.

__2020:__ Two cataclysmic events occurred in late 2019; A Chinese dude ate a bat sandwich and I got a new flatmate. My flatmate didn't enjoy burgers as much as I did, which meant a drastic reduction in Byron + Shake Shack. Also, the pandemic seemed to affect Byron particularly badly (If Lord Byron is reading this, check out Section 3 for some ideas to increase business). The preponderance of Busaba can be attributed to my flatmate, while Capeesh was all mine. We both got behind the Athenian (Halloumi Souvlaki), Manjal (Uthappa aka Savory South Indian rice pancakes) and Scarpetta (Pasta for her, grilled chicken+veggies for me). Due to the lockdown, Dishoom finally entered the food delivery game in the later part of the year. Another pandemic baby was Chowpatty, an upstart, home-run 'restaurant' that delivered Bombay street food (Complete with raw mango garnish).

## Look Ma! Bar Charts!

Honestly, this section is just me flexing my newly developed plotting muscles...

    
![png](/images/2020-05-17/output_42_0.png)
    

Unsurprisingly, most of the food is ordered on the weekends. But why stop here? Let's take a look at:


![png](/images/2020-05-17/output_44_0.png)
    


Another anti-surprise, most of the food ordered during lunch and dinner, with more dinner orders than lunch orders. And because we live in an age of cheap compute, I decided to also plot...


    
![png](/images/2020-05-17/output_46_0.png)
    


OHMYGODOHMYGODOHMYGOD!!! No orders during the [23rd](https://en.wikipedia.org/wiki/The_Number_23) minute!!! Clearly I'm [living in a movie](https://en.wikipedia.org/wiki/The_Truman_Show)...

## Cost Analysis

As calculated before, my total spend on Deliveroo so far is __£5212.37__, which works out to __£87 per month__ on average. 

### Distribution of Order Values

The average price of an order is £16.49 and the median is £15.77. I've plotted the distribution below, and it's pretty obviously multimodal - For eg. The large concentration of orders at about £8 corresponds to the dessert orders. One can also see the larger orders when I ordered for a group of people (Seems like there were at least 7/8 such occasions).


    
![png](/images/2020-05-17/output_49_0.png)
    


### Is my Deliveroo Plus Account worth it?

This calculation is very hard to do super accurately, since the rules of the game keep changing. For eg. in Jan 20, Deliveroo introduced a £10 minimum order value to avail free delivery. The delivery fee structure is also pretty complicated, though Deliveroo states that on average, it is about £2.5. I have no idea how that number has changed over time though - Most of my email receipts before subscribing to Plus state the delivery fee is £0, which makes me wonder why I subscribed in the first place. To cut a long story short I will proceed with the following assumptions - Delivery fee would have been £2.5 without Plus, and the £10 threshold applies to all orders.

I started my Plus subscription in October 2019. At the time, it cost £7.99 per month (Increased to 11.49 per month in December 20). Over 15 months, this is a total spend of £123.35.

Over the same period of time, I had 80 orders that cost more than £10. This translates to savings on delivery fees of £200, and __net savings of £66.65__ (So close!) or a princely sum of __£4.44 per month__. At the present cost of Plus, the net savings would be __less than £2 per month__.

It would be nice if Deliveroo themselves could do this calculation for you. Obviously, they wouldn't want to show you that info if it turns out your Plus membership is actually a net negative for you, and I don't think it would be acceptable for them to only show this statistic to people who benefitted from Plus. 

Hence, the only way such a feature could work is if they showed people who don't have Plus how much they would have benefitted with Plus given their order history - "If you had signed up for Plus in Oct 2019, and used the savings to purchase long-dated Gamestop options, today you'd have a 1000 Dogecoins!". Of course, this is assuming that Deliveroo is actually incentivized to convert such customers to Plus; I've no idea how the economics on that works.

### Most Expensive Restaurant

This should be an easy one, right? The most expensive restaurant is the one that costs the highest price per unit of food, which seems straightforward enough. Unfortunately, the hard part of using that formula is defining a 'unit of food'. Do we define an order to constitute 1 unit? This doesn't seem right, as we will see in the analysis of individual restaurants, order sizes can be quite variable even for orders from the same restaurant (Due to guests for example). 

How about defining a unit of food to be one restaurant item? This is even more problematic, because items can fall into various categories - Mains, Sides, Pizzas (Yes, pizza is a separate category!), Drinks, Desserts etc. and are hence even less uniform than orders. There might be a path here if we manage to categorize the items, and take some sorted of weighted sum across categories (1 Side = 0.5 Units, 1 Main = 1 Unit, 1 Pizza = 1.5 Units etc.) but the categorization is a challenge in it's own right (That I explore in the restaurant level analysis). 

Another approach would be to go full Physics and define food units in calories, but caloric information doesn't exist for most of these items - Hit me up if you can think of a not-too-hard, non-manual way to come up with approximate calorie counts for the items! Also,  caloric count isn't proportional to satiety - The calorie approach would be biased against desserts, but that is just an argument for why desserts should be their own category.

In the absence of clear answers, I decided to just calculate the 2 simplest metrics (Average Order Value and Average Item Value), and see which one made more sense.



<table border="1" class="dataframe">
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


Off the bat, we can see that Item Value as a metric is giving nonsensical results - The top 3 items have Pizza restaurants (As predicted) and Motu Indian Kitchen, which has the massive "Box for 1" as a single item, but is hardly an expensive/high class restaurant. The other entries on the list are equally nonsensical (The Athenian? Rusty Bike?? Cookies & Cream???)




<table border="1" class="dataframe">
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


This seems more in line with the truth, but again there is a very evident bias - All of Dishoom, Manjal and PizzaExpress have instances of large (> £60) orders, which skew their average order value upwards. 

Another point in favour of the order metric - This list features the fancier places like Ping Pong, Busaba, and Byron higher up than the first list.

# Restaurant Level Analysis

In this section, I want to break down my orders from some of my favourite restaurants and see which items I ordered most, and provide some commentary.




## Axes of Interest

What kind of information are we interested in when looking at a particular restaurant?

1. Summary stats: Total number of orders? Total spend on the restaurant? How expensive is the restaurant? How often do I order from the restaurant?

2. I'd like to see a breakdown of the summary stats over time, since averages don't always tell the whole story. There are two graphs here: 
    - __Histogram of order values:__ This gives a fair idea of what ordering behaviour was for a particular restaurant - Did I always have the same standard order? Did I order from this restaurant when entertaining guests? 
    - __Chart of order frequency:__ Each time I ordered from that restaurant, I plotted it's order share in the last 10 orders. This was benchmarked against 1/(Number of distinct restaurants in last 10 orders) - The logic being that if all restaurants were equally popular, the value of order frequency would be the benchmark value.


3. What are the most popular items for this particular restaurant? I've also included some personal commentary in this section.

## Sidebar: Clustering

Ever since I decided to try to want to become a datadude, I've had an irresistible urge to cluster things together and this seemed like the perfect opportunity. In this sidebar, I describe two different problems that came up in the analysis, and how I approached them with clustering.

### Item Categorization - K-Means Clustering

As discussed before, it's handy to have some way to categorize restaurant items into Mains/Sides/Drinks. It's a useful axis to view the data along (Most popular Main?), and can also feed into more complicated analyses (How wasteful are you being when you order drinks from the restaurant instead of buying them from the supermarket below your house?)

Probably the most accurate way to do item categorization is to somehow scrape the Deliveroo menus for each restaurant, and look up which category they fall into. I do not have the coding skills required to do this, and even if I did, it probably wouldn't be as straightforward (Item names change over time, as we will discuss below). 

If I can't get the data from an external source, I have to look within. The only information I have per item is it's name and it's price (Well, I also know which other items typically accompany it, but it's much harder to extract meaningful inferences from that kind of data, particularly when your dataset is so small). I don't know how to extract the category data from the item name, and so I decided to go with a simple binary classification of items per restaurant into Mains or Sides, based on their price. What this boils down to is applying K-Means clustering to a 1-dimensional dataset (The list of prices in this case). I don't know if K-Means is ideal for such situations, or if there are simpler and more direct methods one can leverage, but it was available in SKlearn library and seemed to give good enough results.

For certain restaurants this kind of binary categorization doesn't make sense - For eg. Ping Pong has a system of small plates, so all their items are Mains (or Sides). In such a case, forcing a divide into 2 categories creates an artificial distinction. There are ways to determine the correct number of clusters for a dataset, but I don't have a lot of expertise in these matters and hence decided to just take this as an input from the user.




### Item Name Changes - Prefix Clustering

When analysing the distribution of items per restaurant, I found that items names will sometimes change over time. Example: Shake Shack will sometimes refer to it's "Chipotle Cheddar Chick'n Burger" as just "Chipotle Cheddar Chick'n", and the name of Ping Pong's "Potato and Edamame Cake __(V)__ (2pcs)" changed one day to "Potato and Edamame Cake __(v)__ (2pcs)". These name changes lead to some nasty surprises; Can you imagine my horror when I saw "Garden Fresh Golden Dumplings" topping the Ping Pong list instead of my beloved cakes of potato and edamame?!

This is similar to the issue I faced earlier in the analysis with restaurant names, but solving this with manual relabelling is far more tedious, since the universe of items is much larger. 

My first thought was to define the distance between two items as the [edit distance](https://en.wikipedia.org/wiki/Edit_distance), connect all points (i, j) in the resulting graph with distance(i, j) <= d (For some suitable d) and then each connected component would be defined as a separate cluster. You could then define some appropriate point inside the cluster as the representative (The centroid maybe?) However, it didn't feel like this problem was worth that much effort.

The next version of this idea was to draw a directed edge from i -> j if lowercase(i) is a prefix of lowercase(j). If lowercase(i) == lowercase(j), we use a lexicographic ordering to break a tie and avoid cycles. This sets up a directed acyclic graph over the items, and we can again define connected components as clusters. The lowest common prefix of all the strings in the cluster pretty much presents itself as the natural candidate for cluster representative.

This kind of clustering seems to work quite well for my dataset and didn't seem to cluster any distinct items together. However, as a future improvement, it might be a good idea to also consider the average item price as a co-ordinate when evaluating the distance - If the item is the same, the price will be similiar as well.



## Shake Shack

Shake Shack is my favourite restaurant, and with 47 orders, the data supports this. I first discovered it in the US and fell in love with their cheesy fries (As we will see, the data supports this as well).


### Summary Stats

__Consumption:__


Total Number of Orders:  47


Total Value of Orders:  707.4


__Cost:__

Average Order Value:  15.05

Median Order Value:  17.45


__Order__ __History:__

First Order: 2018-11-22

Last Order:  2020-12-21

Order Frequency:  1.86  per month
    
    
![png](/images/2020-05-17/output_68_1.png)
    


### Item-wise Analysis

Next up I looked at the distribution of the items.


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
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: left;">
      <th>Item</th>
      <th>Qty</th>
      <th>Value</th>
      <th>Avg Price</th>
      <th>Item Type</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Crinkle Cut Fries with a pot of Cheese Sauce (V)</th>
      <td>45</td>
      <td>180.00</td>
      <td>4.00</td>
      <td>Side</td>
    </tr>
    <tr>
      <th>Shack Cheese Sauce</th>
      <td>21</td>
      <td>21.00</td>
      <td>1.00</td>
      <td>Side</td>
    </tr>
    <tr>
      <th>Chocolate</th>
      <td>21</td>
      <td>124.95</td>
      <td>5.95</td>
      <td>Main</td>
    </tr>
    <tr>
      <th>Chick'n Shack</th>
      <td>16</td>
      <td>120.00</td>
      <td>7.50</td>
      <td>Main</td>
    </tr>
    <tr>
      <th>Chipotle Cheddar Chick'n</th>
      <td>14</td>
      <td>119.00</td>
      <td>8.50</td>
      <td>Main</td>
    </tr>
    <tr>
      <th>Black Truffle Chick'n</th>
      <td>7</td>
      <td>66.50</td>
      <td>9.50</td>
      <td>Main</td>
    </tr>
    <tr>
      <th>Fries (V) (VG)</th>
      <td>5</td>
      <td>15.00</td>
      <td>3.00</td>
      <td>Side</td>
    </tr>
  </tbody>
</table>
</div>


As is evident, I am a huge fan of the cheesy fries, and an even bigger fan of the cheese sauce that they serve along with it - Counting the default serving you get with the fries along with the extra servings, that's 76 pots of cheese sauce! This makes sense, because having to ration cheese sauce (Or worse, share it) is an abominable thought. If there was ever any doubt, I'd order an extra serving. At a price of £1, it was a steal - Though the caloric cost (240 kcal) is much higher.

The K-Means clusterer has categorized the Chocolate Shake as a Main based on the £5.95 price point, but given that this is __Shake__ Shack we're talking about, it's probably OK.

My relation with their burgers has evolved over time; At first I could only eat the Shroom burger, which kinda sucked. I was hugely excited about the launch of their fried chicken sandwich, which did not disappoint (At first). Over time though, it started tasting super dry and chewy, and whenever they came out with a special edition chicken burger (Like the Chipotle Cheddar Chick'n, or the Black Truffle Chick'n) I'd immediately switch loyalties. I'm still not sure why the special edition burgers tasted so much better; My theory about the Black Truffle Chick'n is that it was made of thigh meat, which meant a juicier and more tender patty. While the data doesn't reflect this, recently life came full circle and I switched back to the Shroom burger.

## Byron

Byron clocks in at number 2 in terms of all-time favourites, though I'm not sure when I'll eat there next. The pandemic seems to have hit this chain particularly hard. Most outlets haven't re-opened, and of the ones that have, none deliver to my house. They've also booted my favourite items from their new menu, so don't know if I'll go back at all.

For old time's sake then, here are the Byron stats:

### Summary Stats




Consumption:
	Total Number of Orders:  31
	Total Value of Orders:  679.3
Cost:
	Average Order Value:  21.91
	Median Order Value:  20.65
Order History:
	First Order: 2018-05-16
	Last Order:  2019-11-27
	Order Frequency:  1.66  per month
   
    
![png](/images/2020-05-17/output_74_1.png)
    


### Item-wise Analysis



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
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Qty</th>
      <th>Value</th>
      <th>Avg Price</th>
      <th>Item Type</th>
    </tr>
    <tr>
      <th>Item</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Blue Cheese Sauce</th>
      <td>26</td>
      <td>36.9</td>
      <td>1.42</td>
      <td>Side</td>
    </tr>
    <tr>
      <th>Onion rings (V)</th>
      <td>19</td>
      <td>76.0</td>
      <td>4.00</td>
      <td>Side</td>
    </tr>
    <tr>
      <th>Chocolate</th>
      <td>16</td>
      <td>80.0</td>
      <td>5.00</td>
      <td>Side</td>
    </tr>
    <tr>
      <th>V-Rex + Fries</th>
      <td>14</td>
      <td>190.0</td>
      <td>13.57</td>
      <td>Main</td>
    </tr>
    <tr>
      <th>Classic Chicken</th>
      <td>12</td>
      <td>143.0</td>
      <td>11.92</td>
      <td>Main</td>
    </tr>
    <tr>
      <th>Byron Lager (500ml)</th>
      <td>8</td>
      <td>47.6</td>
      <td>5.95</td>
      <td>Side</td>
    </tr>
    <tr>
      <th>Sriracha Mayonnaise (V)</th>
      <td>4</td>
      <td>5.8</td>
      <td>1.45</td>
      <td>Side</td>
    </tr>
  </tbody>
</table>
</div>


The item distribution is almost exactly the same as Shake Shack (Fries don't feature on the list, as Byron would provide them with the burger).

Cheese Sauce at the top of the pile again! Dunking their large, messy beer-battered onion rings into the Blue cheese sauce, adding a touch of mustard and then devouring the result was a ritual I'd perform every single time. The Byron Chocolate Shake was also amazing (Much better than Shake Shack).

On the burgers, I actually really like their classic grilled chicken burger, both in terms of the flavour, and the health angle - It was the leanest burger among all the burgers in this analysis. The V-Rex was a special edition vegetarian burger, that is probably the closest I've seen UK veg burgers come to the veg burgers we have in India (It had a crunchy deep-fried patty, spicy mayo and and a slice of onion in it).

Speaking of the V-Rex reminded me of one my biggest pet peeves - Why isn't anyone making veg burgers out of potatos? If there are any English restaurant owners reading this, ditch the halloumi/beans/jackfruit/Beyond Meat patties, and use potatos instead!! I don't know why no one has come up with this yet, but a smattering of vegetables in a matrix of mashed potatos, breaded and deep-fryed, results in the tastiest vegetarian burgers, and if you put this on your menu, you will win a lot of business and goodwill from the large and rapidly growing Indian immigrant community.

## Ping Pong

Ping Pong was the first restaurant I ever ate at in London. I've always enjoyed momos, springrolls and assorted dimsums, and Ping Pong elevated that whole experience into something very special. Unfortunately I only lived within the Ping Pong delivery radius for a small window in space-time, otherwise it would be a serious contender for the top spot.

### Summary Stats



Consumption:
	Total Number of Orders:  12
	Total Value of Orders:  238.25
Cost:
	Average Order Value:  19.85
	Median Order Value:  19.95
Order History:
	First Order: 2019-01-26
	Last Order:  2019-05-09
	Order Frequency:  3.5  per month
    
    
    


    
![png](/images/2020-05-17/output_79_1.png)
    


### Item-wise Analysis


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
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Qty</th>
      <th>Value</th>
      <th>Avg Price</th>
    </tr>
    <tr>
      <th>Item</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Potato and Edamame Cake (v) (2pcs)</th>
      <td>12</td>
      <td>41.40</td>
      <td>3.45</td>
    </tr>
    <tr>
      <th>Mixed Vegetable Spring Roll (v) (3pcs)</th>
      <td>8</td>
      <td>30.00</td>
      <td>3.75</td>
    </tr>
    <tr>
      <th>Spicy Vegetable Dumpling (v) (gf) (3pcs)</th>
      <td>8</td>
      <td>30.80</td>
      <td>3.85</td>
    </tr>
    <tr>
      <th>Golden Dumpling (v) (gf) (3pcs)</th>
      <td>7</td>
      <td>26.25</td>
      <td>3.75</td>
    </tr>
    <tr>
      <th>Vegetable Sticky Rice (v)</th>
      <td>6</td>
      <td>30.90</td>
      <td>5.15</td>
    </tr>
    <tr>
      <th>Chinese Vegetable Spring Roll (V)(VG)</th>
      <td>4</td>
      <td>15.00</td>
      <td>3.75</td>
    </tr>
    <tr>
      <th>Spinach and Mushroom Dumpling (3pcs) (VG) (GF)</th>
      <td>4</td>
      <td>15.40</td>
      <td>3.85</td>
    </tr>
    <tr>
      <th>Spicy Chinese Vegetable Dumpling (V) (VG) (GF) (3pcs)</th>
      <td>3</td>
      <td>11.55</td>
      <td>3.85</td>
    </tr>
    <tr>
      <th>Asahi</th>
      <td>2</td>
      <td>9.30</td>
      <td>4.65</td>
    </tr>
  </tbody>
</table>
</div>


The Ping Pong item list shows some of the limitations of the prefix clustering approach used to cluster item names. The "Mixed Vegetable Spring Roll" and "Chinese Vegetable Spring Roll" are the same, as are the "Spicy Vegetable Dumplings" and the "Spicy Chinese Vegetable Dumplings". On making these corrections, it looks like I ordered the potato cakes and spring rolls 12 times in 12 orders (And spicy veg dumplings 11 times), so pretty much every time I ordered. The uniform order hypothesis is also supported by the order value histogram.

Not much too say about these items; Just extremely tasty, high quality dimsums that always left me happy and satisfied. How I wish they'd open up a few [dark kitchens](https://www.bbc.co.uk/news/business-47978759)!

The only thing that irritates me about this table is that I paid £9.30 for two bottles of Asahi, when I could've bought 4 bottles from Tesco for £5.50. Paying for drink markups is understandable when you're dining at the restaurant, but it's foolish when you get food delivered at home!

## Rusty Bike

Rusty Bike was my go-to place when I first moved to London. The food was too extravagant/unhealthy, reasonably priced and quite tasty.

### Summary Stats



Consumption:
	Total Number of Orders:  21
	Total Value of Orders:  346.1
Cost:
	Average Order Value:  16.48
	Median Order Value:  14.15
Order History:
	First Order: 2018-07-27
	Last Order:  2020-09-23
	Order Frequency:  0.8  per month
    
    
    


    
![png](/images/2020-05-17/output_84_1.png)
    


### Item-wise Analysis


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
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Qty</th>
      <th>Value</th>
      <th>Avg Price</th>
      <th>Item Type</th>
    </tr>
    <tr>
      <th>Item</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Green Curry</th>
      <td>21</td>
      <td>204.85</td>
      <td>9.75</td>
      <td>Main</td>
    </tr>
    <tr>
      <th>Steamed Rice</th>
      <td>15</td>
      <td>42.85</td>
      <td>2.86</td>
      <td>Side</td>
    </tr>
    <tr>
      <th>Vegetable Spring Rolls</th>
      <td>12</td>
      <td>54.20</td>
      <td>4.52</td>
      <td>Side</td>
    </tr>
  </tbody>
</table>
</div>


As is evident from the price histogram, Rusty Bike orders followed a standard template - Green Curry, Steamed Rice, and on occassion, spring rolls. It is a bit odd that the numbers don't line up better - I'd have expected the Green Curry and Rice orders to be almost equal. On reviewing the raw orders data, I found that initially (For the first six orders), Rusty Bike used to include a default steamed rice with the green curry. They later decoupled them into separate items. One result of this is that the Average Price of the green curry in the table above is inflated - The actual price of the green curry is about £7.5.

## The Pizza Room

I have eaten a lot of pizza in my life and The Pizza Room is something special.

### Summary Stats



Consumption:
	Total Number of Orders:  18
	Total Value of Orders:  419.54
Cost:
	Average Order Value:  23.31
	Median Order Value:  20.94
Order History:
	First Order: 2018-08-10
	Last Order:  2020-11-25
	Order Frequency:  0.64  per month
    
    
    


    
![png](/images/2020-05-17/output_89_1.png)
    


### Item-wise Analysis




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
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Qty</th>
      <th>Value</th>
      <th>Avg Price</th>
      <th>Item Type</th>
    </tr>
    <tr>
      <th>Item</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Quattro Formaggi</th>
      <td>20</td>
      <td>299.20</td>
      <td>14.96</td>
      <td>Main</td>
    </tr>
    <tr>
      <th>Brownie with Ice Cream</th>
      <td>9</td>
      <td>56.25</td>
      <td>6.25</td>
      <td>Side</td>
    </tr>
    <tr>
      <th>Coke 330ml</th>
      <td>3</td>
      <td>7.80</td>
      <td>2.60</td>
      <td>Side</td>
    </tr>
    <tr>
      <th>Margherita</th>
      <td>2</td>
      <td>24.30</td>
      <td>12.15</td>
      <td>Main</td>
    </tr>
  </tbody>
</table>
</div>


Wherever I go, the Quattro Formaggi or 4 Cheese pizza has been my go-to pizza order for a while. What makes the Pizza Room Quattro Formaggi special is:

1. __Tomato Sauce:__ Nowhere else have I seen Pizza Room levels of clarity on this topic. They are upfront about the fact that the default option is no tomato sauce (Traditionally, the QF is a 'White Pizza'). However, if you'd like tomato sauce, they will add it on for a fee. I really like the fact that they charge me a nominal amount for the sauce and hence eliminate all uncertainty - At other restaurants I have to add a delivery note saying "If you don't typically add tomato sauce to the Quattro Formaggi pizza, please can you do so in this case?" and then pace nervously till the pizza gets delivered.

2. __Add-Ons:__ Definitely add green chillies to your QF pizza for a zingy complement to all the cheese.

3. __Generous deposits of Gorgonzola:__ I've no idea why, but a lot of restaurants scrimp on the blue cheese. The Pizza Room isn't one of them.

Again, we see the idiocy of paying >3x for a can of Coke, that could've been fetched from my refrigerator.

## Motu Indian Kitchen

Motu translates to "Fatty" or "Fatboy", and they have shipped me a lot of calories. 

### Summary Stats



Consumption:
	Total Number of Orders:  32
	Total Value of Orders:  565.75
Cost:
	Average Order Value:  17.68
	Median Order Value:  17.5
Order History:
	First Order: 2018-12-08
	Last Order:  2020-01-16
	Order Frequency:  2.38  per month  
    

    
![png](/images/2020-05-17/output_94_1.png)
    


### Item-wise Analysis

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
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Qty</th>
      <th>Value</th>
      <th>Avg Price</th>
      <th>Item Type</th>
    </tr>
    <tr>
      <th>Item</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Box for 1</th>
      <td>28</td>
      <td>506.0</td>
      <td>18.07</td>
      <td>Main</td>
    </tr>
    <tr>
      <th>Tadka Dal (VG)</th>
      <td>9</td>
      <td>31.5</td>
      <td>3.50</td>
      <td>Side</td>
    </tr>
    <tr>
      <th>Pilau Rice (V)</th>
      <td>7</td>
      <td>17.5</td>
      <td>2.50</td>
      <td>Side</td>
    </tr>
  </tbody>
</table>
</div>


On paper, the Box for 1 had everything I could want from an Indian meal: Paneer, Dal Tadka, garlic naan and papads. The quality was not amazing, but it wasn't too bad, and beggars can't be choosers/

Eagle-eyed readers will notice that this table is not consistent with the order value histogram above - The Box for 1 costs £18.07, but there are no orders with that price. Maybe the price increased over time? But in that case you'd probably have a bimodal distribution with two peaks. What gives? In order to crack this one, I had to go back all the way to the email receipts.

Turns out Motu will let you add extra food to your Box for 1 order, and the Deliveroo email receipt lists it all under the Box for 1. I didn't account for this case while writing my email parser (And I'm not sure how I should, since the unit prices of the individual items aren't given). Hence the £18.07 is a an average of the vanilla, no-frills-attached Boxes for 1 and the fancier, with-extra-fixin's Boxes for 1, like the one below.

![png](/images/2020-05-17/Motu_cornercase.PNG)

## Dishoom

A lot of people love to shit on Dishoom. These are the fools that equate contrarianism with intelligence. Dishoom is fucking amazing, and I'll fight anyone who says otherwise. 

### Summary Stats



Consumption:
	Total Number of Orders:  6
	Total Value of Orders:  166.2
Cost:
	Average Order Value:  27.7
	Median Order Value:  21.9
Order History:
	First Order: 2020-08-14
	Last Order:  2020-12-19
	Order Frequency:  1.42  per month
    
    
    


    
![png](/images/2020-05-17/output_100_1.png)
    


### Item-wise Analysis

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
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Qty</th>
      <th>Value</th>
      <th>Avg Price</th>
      <th>Item Type</th>
    </tr>
    <tr>
      <th>Item</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Garlic Naan (V)</th>
      <td>10</td>
      <td>35.0</td>
      <td>3.50</td>
      <td>Side</td>
    </tr>
    <tr>
      <th>House Black Daal (V)</th>
      <td>6</td>
      <td>51.9</td>
      <td>8.65</td>
      <td>Main</td>
    </tr>
    <tr>
      <th>Chilli Chicken</th>
      <td>4</td>
      <td>27.6</td>
      <td>6.90</td>
      <td>Side</td>
    </tr>
    <tr>
      <th>Pau Bhaji (V)</th>
      <td>1</td>
      <td>5.7</td>
      <td>5.70</td>
      <td>Side</td>
    </tr>
    <tr>
      <th>Chole (Ve)</th>
      <td>1</td>
      <td>9.5</td>
      <td>9.50</td>
      <td>Main</td>
    </tr>
    <tr>
      <th>Mattar Paneer (V)</th>
      <td>1</td>
      <td>11.5</td>
      <td>11.50</td>
      <td>Main</td>
    </tr>
    <tr>
      <th>Chicken Berry Britannia</th>
      <td>1</td>
      <td>12.5</td>
      <td>12.50</td>
      <td>Main</td>
    </tr>
    <tr>
      <th>Chicken Ruby</th>
      <td>1</td>
      <td>12.5</td>
      <td>12.50</td>
      <td>Main</td>
    </tr>
  </tbody>
</table>
</div>


Dishoom has the best naan I've had in the UK, just by virtue of the fact that it's actually a naan, and not an insipid cloud of semi-cooked dough. The black daal is among the best I've had anywhere, but my favourite item has to be the Chilli Chicken, which is basically popcorn chicken tossed in a sticky, spicy sauce. Dishoom, if you guys are reading this, how about showing vegetarians some love and adding Chilli Paneer to the menu?

# Next Steps

The following are some ideas for next steps:

1. Add support for UberEats: I'm primarily a Deliveroo user myself, but I did go through an UberEats phase. Would be nice to get that data in here as well.

2. Calorie counts: I would really like to be able to figure out caloric consumption rates, but I've no idea how I'd go about it.

3. Infer regime changes from the data. As seen in the analysis above, the data often changed due to events taking place in my life and the world in general. I would like to be able to automatically infer the different regimes from the data in some way. In a high-level, abstract sense, the way to do this would be to fit some kind of model to the data in the current phase, and calculate the probability of the next phase occurring given that fit. If the probability is low, that means an event has occurred which caused the fit to change. I have no idea what such a model would actually look like though.

4. Correlate this with other datasets in my life: Was I on a health kick when I didn't order in those 2 weeks in Jan? Did I order that brownie sundae late on Friday night because I had an extra long day at work? What do I typically watch on TV when eating Shake Shack?

5. Get everyone else's data: Wouldn't it be awesome if everyone in London ran the parser and uploaded their spreadsheets into a central repository? We could mine the resulting dataset for insight into how all of London eats. We could charge restaurant owners and investors to run queries on the dataset, or sell them insights that are mined from it. For eg. "Guys, Chilli Chicken is super popular right now, you gotta add it to your menu!" Or how about "Guys this hole-in-the-wall type place in Brixton is really blowing up, maybe we should invest and hook them up with a location in Central London?". 

    If Deliveroo ever opens up a restaurant consultancy, remember, you saw it here first!

6. Look at the other players in the Deliveroo ecosystem: What about the riders? Would they get some benefit out of analysing their delivery data? Comparing their stats against other riders, or the population average? I don't know how much data Deliveroo shares with them, and in what format, but employers are typically incentivized to give their employees as little information as possible.

