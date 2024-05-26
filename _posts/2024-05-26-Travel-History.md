# Google location history

Last year, I had to submit a large form with my travel history outside of UK. Obviously I didn’t want to sit with my passport and look through 5 years worth of stamps and try and figure out from my inbox when the corresponding flights were. 

Instead, I decided to leverage the location history that Google has been storing for me over these years. Went to google Takeout and downloaded it and I was ready.

**Takeout Choices - Raw or Processed?**

Google gives you your history in 2 formats. 

Format 1 is the raw data, which is basically a giant list of (point in time, point in space). This is the physicist’s worldline - Your life is just a discontinuous curve moving through 3-dimensional spacetime (Google maps doesn’t store your spatial z coordinate. The discontinuities are the times your phone wasn’t online). The great thing about this format is the data schema is as simple as it can get. The bad part is you have to do all the data crunching.

Format 2 is the semantic format. Here Google uses it’s world-class cutting edge algos and armies of highly paid data scientists to slap some meaningful labels on the raw worldline data. At a high level, your location history is divided into placeVisits (Where you spend some time stationary at location X) and activitySegment (Where you travel between location X and Y). The placeVisits may have some information like the address and the the activitySegments may have information like mode of transport etc. 

The problem with these is that the datamodel is complicated and varies over time. So you if you use field x in your analysis, you have no guarantee that it will exist in all the datapoints for your entire timeline or that it will exist in future timeline objects. 

Further, Google will sometimes fuck up the tagging of activitySegments (like the time they thought I cycled from Lucknow to Mumbai in 2 hours)

**Approach 1 - Rawdogging data, Costly API calls, Simplifying assumptions**

Given the above, it seemed that if we want a future proof solution, the most reliable way is to use the raw worldline. Further, for the task I had in mind, the solution seemed super simple - For each point on the worldline, map it to the corresponding country. Find all the points in time when the country changes and boom - You have your travel history table. 

**Problem with this approach**: Mapping from (latitude, longitude) → Country is an expensive function call. The raw data contains a point every 15-20 seconds, so that translates to a lot of points.  In 3000 days of history, I had a 15 million raw data points. The really dumb approach of simply mapping each raw datapoint to a country is way too slow. Maybe we could do something cleverer like form a small number of clusters (1000?), map each cluster to a country and then use that to label raw points? It could work, but it’s not foolproof by any means and relies on a complex algo. I’m a lazy fucker who hates complexity so I decided to think some more before going down this route.

**Exploiting Timeline gaps:** The approach I took was to use the fact that the data is already naturally clustered - As I stated earlier, the worldline is discontinuous whenever my cellphone is not online and these gaps cluster the raw data in a natural way - The insight is to realise that most country changes occur during such gaps. This is because:

1. I mostly travel between countries by flight - This is specially true for travel in and out of the UK.
2. There is a gap in my location history while I’m flying as data connectivity is lost (I never access wifi on the plane, not sure how this would be affected if I did).

Now there will obviously be some discontinuities which were not travel related - Maybe my phone ran out of battery, or I didn’t have connectivity, but that’s OK - By focussing just on the timeline gaps, we massively reduce the number of points we need to reverse map. We can further cut down the points by prioritizing the biggest gaps first.

**Final Approach:** So what I did is evaluate the physical distance between consecutive points in the worldline (I used the Haversine metric, just to feel mathematically literate). Then I reverse-sorted and got the consecutive worldline points with the largest distance between them. Applied some sensible threshold cutoff - 500 kms seems like a good lower bound for international flights. And then I was left with a few hundred  points that I mapped the countries for, and ended up with a list of all my international travels.

**Comments**: This solution works for me (And one other super lazy friend who I sent my notebook to) but will the underlying assumptions hold for everyone? Certainly it won’t capture the country changes for Europeans that routinely drive/take trains across country borders. This is not super satisfying. On the plus side, it uses the raw data, and the code and logic is extremely simple - The only complicated bit is the country lookup and we outsource that to an API.

**Approach 2 - Processed Data only**: 

Chronologically, this was the first approach that I took to solve this problem - In this case I used the semantic data and went through the following steps: 

1. Isolated the activitySegments tagged as FLIGHTS, 
2. Looked at the placeVisits before and after those FLIGHT Segments, 
3. Extracted the airport name from the address of the placeVisits
4. Manually compiled a mapping of which airport was in which country.

This approach was pretty stupid, and I would’ve been better off mapping the start and end locations of the FLIGHT segments using the country lookup. I also spent way too much time at this step figuring out the meaning of random useless fields in the structured data, how they were related to one another and if the data was internally consistent (Was the startTime of each activitySegment AFTER the endTime of the previous activitySegment?)

**Other approaches:**

This idea is obvious enough that a few other people have implemented their own versions of it:

1. [Laurens Geffert](https://janlauge.github.io/2021/google_timeline_travel_history/): One of the aforementioned highly talented Google data scientists - His solution was the optimal mix of my 2 approaches - He whittled down the space of points required by focussing on the processed data and then mapped it using a country lookup. I suspect the API he used to do the country lookups was also much more performant than the one I used (I just used whatever ChatGPT recommended). 
2. [https://geoprocessing.online/](https://geoprocessing.online/pricing/): This company lets you upload your timeline data and then helos you with various analyses. I didn’t want to upload my data anywhere so haven’t experimented with this.
3. [mileagewise](https://www.mileagewise.com/): They use google timeline data to create mileage logs (In the US you can claim tax benefits by claiming the mileage incurred while driving around as a business expense) This is a very different usecase from mine obviously, but still an ingenious idea.