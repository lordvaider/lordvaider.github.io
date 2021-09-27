# Deliveroo Data Analysis I

In this series of posts, my goal is to analyse my Deliveroo order data starting May 2018 and see what I can learn about myself from it.

The series is divided into 3 sections:
 1. __Section I: Fetching the raw data.__ Provides details on the steps taken to parse email order receipts and obtain the data in a structured format.
 2. __Section II: Top level analysis.__ Simple things like spend per year, consumption patterns over time, distribution by cuisine/restaurant and how it changed over time. Also includes a behind the scenes look at why these patterns changed in the way they did. 
 3. __Section III: Restaurant-wise analysis:__ Basic stats and graphs for the most popular restaurants. Also includes a list of the most popular items, reviews and other personal tidbits. 
 
 __Table of Contents:__
 
 * TOC
{:toc}

# Step 0 - Build Mad Skillz

Despite having an advanced degree in Computer Science and working as a sort-of-software engineer for several years, I knew embarrasingly little about the basic machinery required to work on this project. Hence I spent a fair bit of time just learning some super simple stuff:

1. __Pandas:__ For those that don't know, Pandas is a Python library used for dealing with tabular data sets (Dataframes) - Pretty much bread and butter for any datadude. Again, going through the Kaggle tutorial was sufficient to get started.

2. __Jupyter Notebooks:__ I guess the only thing I had to learn was the fact that they exist! I really love it when tools are powerful AND easy to use - I felt like I had been looking for Jupyter notebooks my entire life!

3. __Other tools:__ Played around with a couple of different code editors, picked up the basics of the command line and git and went through some of the lectures [here](https://missing.csail.mit.edu/).  

4. __Touch Typing:__ Imagine having to use a pencil taped to a brick every time you want to write something. That is what using a keyboard felt like to me. I decided to make the interface between my brain and the computer as seamless as possible, and learnt to touch type. Best investment of my time ever!

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

I have shared the end result of these efforts in git repo [here](https://github.com/lordvaider/DataDeliveroo), in case any of you are interested in fetching your own Deliveroo data.

Eventually, I was able to get my data extraction working and got the raw data corresponding to (almost) all orders in a neat .csv file. Just scrolling through this file, I got a feeling of power. <s>Finally it was <b>ANALYSIS!</b> time!</s> Finally, it was time to clean my data and get it into the right format!

## Data Formatting + Cleaning

In order to do useful __ANALYSIS!__ on the data, it first needed some cleaning and formatting: 
1. __Datatype Conversions:__ Need to convert the values of the "Date" column into datetimes.


2. __Add Inferred Columns__: Enrich the dataset with some more columns that will be required in the course of the analysis such as Value, and OrderNo - All items ordered at the same time share an order no.


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

In order to see the top level analysis, check out [Section II](https://lordvaider.github.io/2021/05/18/Deliveroo-Analysis-II.html)!

{% include comments.html %}
