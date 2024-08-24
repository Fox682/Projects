# Sock Sort
-----------------------

## Abitrarily Sorting Socks

This algorithim doesn't technically sort things, but it behaves kind of like sorting a large pile of socks, but without mating them. 

### The "Sock Sort" Analogy:

#### Sorting Socks: 
We sort "socks" (data values) by grabbing them one by one, placing them in piles with their matches. This function sorts data by counting and organizing values as they are encountered.

#### Counting: 
As you grab each "sock" (data value), you check if you've seen a similar one before (just like checking if you've already made a pile for that type of sock). If so, you add it to the pile (increment the count). If not, you start a new pile (add the value to your list and set its initial count).

#### Order Preservation: 
The order in which you first encounter each type of "sock" is preserved, this function keeps track of the order in which each unique value is first seen.

Code:

```python3
#!/usr/bin/python3

import random
import time
import statistics

baskets = []
socks = []

#Calculate run time of the Sorting Algorithim
st = time.process_time() #Start time!

# Create random Socks (numbers/objects)
# Simple random number generator with a range
def sock(x, y):
        randnum = random.randint(x, y)
        return int(randnum)

#### Sock Sort! ####
# IF we don't know how many baskets we will need
# Sort the random "socks" into arbitrary baskets
# label the baskets with a count of socks next in list
# ['Sock Type', 'count']
def sortsocks():
        count = 0
        a = 0
        while count < 1000000: #1 million "Socks"
                count += 1
                a = sock(0, 255)
                b = 0
                if str(a) in baskets: #If this sock exists in basket list
                        #print("Putting sock in basket!")
                        b = baskets.index(str(a))  # Get the index in basket
                        baskets[b + 1] += 1  #Add the sock count to basket
                elif str(a) not in baskets: #If not, create bucket for unique
                        #print("Creating new basket!")
                       baskets.append(str(a)) #Label for Basket
                       baskets.append(1)      #Count This Sock

# Gather only the counts of the socks
# Put them in a list
# For use in calculating Standard Deviation for Random
def pullsocks(basketlist):
        global socks #Sock counts are here
        for i in basketlist:
                if isinstance(i, int):
                        socks.append(i)
                if isinstance(i, str):
                        ...

########################

#Run the sort!
sortsocks()
et = time.process_time() #We're done generating, go spit out stats

#Results!
print("Baskets! ", baskets)

# Stats are generated here
pullsocks(baskets)
#print("Socks! ", socks)
std_dev = statistics.stdev(socks)
print(f"Standard Deviation: {std_dev}")

# Get execution time
res = et - st
print('CPU Execution time:', res, 'seconds')
```

## Performance of the Algorithim 
-----------------------------
The chart below shows the performance of the algorithim 'x' axis is time (in seconds) 'y' is the number of buckets 100,000 items were sorted into.

![Algorithim Performance Chart](https://github.com/Fox682/Projects/blob/master/AlgoPerform.png)



