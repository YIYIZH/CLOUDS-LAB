# Cloud_PigLab

Yiyi ZHANG & Linxia GONG

## Exercise 1:: Word Count

### Questions:

- **Q1: Compare execution times between your Pig and Hadoop MapReduce jobs: which is faster? What are the overheads in Pig?**
  - Hadoop Mapreduce has better performance but require more develop time. Pig is short and quicky written, optimizations are already done automatically. But it supports limited set of operations, while hadoop mapreduce can be controlled and designed be developpers.
- **Q2: What does a GROUP BY command do? In which phase of MapReduce is GROUP BY performed in this exercise and in general?**
  - Group by: Group rows by one or more keys.
  - In map phase it sets the key for group, here the key is each word. The Reducer phase processes collection of values for each key and write them to the disk. Here it does the grouping and sorting by word.
- **Q3: What does a FOREACH command do? In which phase of MapReduce is FOREACH performed in this exercise and in general?**
  - FOREACH: iterates row-by-row through the whole dataset to performs operations.
  - In reduce phase FOREACH is used. It is used to count the frequncy (sum the values) for each word(key).

## Exercise 2:: Working with Online Social Networks data

### Questions:

- **Q1: Is the output sorted? Why?**

  Yes. *Group by* clause collects all records with the same value for the provided key together into a bag, and in the schema keys are declared as long numbers. During the reduce phase of the job, keys (with their bags) are sorted by default sorter.

- **Q2: Can we impose an output order, ascending or descending? How?**

  Yes, with *order by* clause, use `order Output by NumberOfFriends (ASC|DESC)`. Here ASC and DESC determine the sort order.

- **Q3: Related to job performance, what kinds of optimization does Pig provide in this exercise? Are they useful? Can we disable them? Should we?**

  Optimazation: multiquery execution. The objective of this feature is to minimizes the I/O: try to read the input once, in single job.

  To turn it off: `pig -x local -M script.pig`, `pig -x local -no_multiquery script.pig`. But we shouldn't disable it.

- **Q4: What should we do when the input has some noise? For example: some lines in the dataset only contain USER_ID but the FOLLOWER_ID is unavailable or null.**

  Use the filter clause to filter null values, as `FILTER Data BY FOLLOWER_ID is not NULL`.



## Exercise 3:: Find the number of two-hop paths in the Twitter network

### Questions:

- **Q1: What is the size of the input data? How long does it take for the job to complete in your case? What are the cause of a poor performance of a job?**

  - Size: 11.7 Mb, 904215 records
  - Big? not so big.
  - How long: Pig script completed in about 30 minutes.
  - Causes:
    Cluster load
    I/O (multiple jobs)
    Join operation requires shuffling data alot

- **Q2: Try to set the parallelism with different number of reducers. What can you say about the load balancing between reducers?**

  - Some reducers have to work as 2-3 times as the other reducers (not good).

- **Q3: Have you verified your results? Does your result contain duplicate tuples? Do you have loops (tuples that points from one user to the same user)? What operations do you use to remove duplicates?**

  - There are no duplicate tuples and no loops. We use the DISTINCT operation.

- **Q4: How many MapReduce jobs does your Pig script generate? Explain why**

  - There are 2 jobs. With MultiQueryOptimizor feature, the pig script generate only 2 jobs instead of 4, which is the expected number of jobs (Load dataset A, Load dataset B, Join operation, Distinct operation).

    ​

## Use-case:: Working with Network Traffic Data

### Exercise 1

#### 1/A “Network” Word Count 

> count the number of TCP connections per each **client** IP.

Group the input with key "client_ip", use COUNT to generate connection counts for each client IP as output.

#### 1/B

> count the number of TCP connection per each IP.

1. **How is this exercise different from Ex. 1?**

   It needs to count the connection numbers of both client IP and server IP. 

2. **How would you write it in Java?**

   Map: for every record, emmit 2 tuples (client_ip and server_ip)

   Reduce: for each key, count values.

3. **Elaborate on the differences between Ex.1 and Ex.1/b**

   Different tuples to generate as intermediate data.

   ex.1b: First we generate by client_ip, then generate by server_ip, use UNION to get a record with both client_ip and server_ip. Choose $0 from the record as key, use COUNT to get values for each key(client_ip or server_ip).

#### 1/C

> count the number of TCP connection per each client IP, at each of the following time granularities: hour, day, week, month, year...

With `CUBE data BY ROLLUP(client_ip, year, month, week, day, hour)`, we get the result with the combination of rollingup dimensions above.

### Exercise 2

> count the total number of TCP connection having, e.g., “google.it” in the FQDN (Fully Qualified Domain Name) field (field #113 in the tstat data).

With "FILTER" operatior, we get the result that there is no record found with “google.it” in the FQDN field.

### Exercise 3

>  for each client IP, compute the sum of uploaded, downloaded and total (up+down) transmitted bytes.

Output in the format of "client_ip, sum_of_uploaded_bytes, sum_of_downloaded_bytes, sum_of_total_bytes". For example:

```
210.240.202.173 1342 20864 22206
```

### Exercise 4: find the top 100 users per uploaded bytes

**Questions:**

1. **Is this job map-only? Why? Why not?**

   No. This job uses some operators that start a reduce phase, such as *GROUP, TOP*.

2. **Where did you apply the TOP function?**

   After we count the uploaded bytes for every user. With the user-bytes relation, we apply the TOP function.

3. **Can you explain how does the TOP function work?**

   TOP function will sort the data with descending order first, then choose the first n tuples that we want.

4. **The TOP function was introduced since PIG v.0.8. How, in your opinion, and based on your understanding of PIG, was the query answered before the TOP command was available? Do you think that it was less efficient than the current implementation?**

   Instead of TOP function, we can use *ORDER BY* and *LIMIT* to achieve the desired result. And it's less efficient because all the records of ORDER function need to flow through the pipeline before they are chosen, it will have greater overhead.

## Use-case: Working with an Airline dataset

### Exercises:

In the following, we propose a series of exercises in the form of Queries.

#### Query 1: Top 20 airports by total volume of flights

> What are the busiest airports by total flight traffic. JFK will feature, but what are the others? For each airport code compute the number of inbound, outbound and all flights. Variation on the theme: compute the above by day, week, month, and over the years.

We observe output in the form of (month, airport code, traffic). 

Airport ATL is the busiest airport by total flight traffic in all the months in the year 2008. With the kownledge about Atlanta international airport, we know that even though Atlanta is not a super city, but it's the city which connects USA and other countries. Most passangers choose to transfer here.

#### Query 2: Carrier Popularity

> Some carriers come and go, others demonstrate regular growth. Compute the (log base 10) volume -- total flights -- over each year, by carrier. The carriers are ranked by their median volume (over the 4 year span).

We observe output in form of "year, carrier_name, popularity".

| Carrier | 2005  | 2006  | 2007  | 2008  |
| ------- | ----- | ----- | ----- | ----- |
| WN      | 6.015 | 6.041 | 6.067 | 6.079 |
| AA      | 5.828 | 5.808 | 5.802 | 5.782 |
| DL      | 5.818 | 5.704 | 5.677 | 5.655 |

We can see that the usage of DL is reducing, while WN demestrates regualr growth, usage of AA is stable with slight reduction.

#### Query 3: Proportion of Flights Delayed

> A flight is delayed if the delay is greater than 15 minutes. Compute the fraction of delayed flights per different time granularities (hour, day, week, month, year).

We use record of year 2005 to analyse by (month, weekday). WE can find July, Auguest and December are the months with greatest delay comparing to other months. It's because of the holiday there are more flights, and the probability of delay is also bigger. 

#### Query 4: Carrier Delays

> Is there a difference in carrier delays? Compute the proportion of delayed flights by carrier, ranked by carrier, at different time granularities (hour, day, week, month year). Again, a flight is delayed if the delay is greater than 15 minutes.

We use the data from 2008.csv. In 2008, the carrier with the most proportion of delay is OH with 0.29. As for the WN we talked in previous question, it only has 0.19 of delays, that may be a reason that it becomes more and more popular.

#### Query 5: Busy Routes

> Which routes are the busiest? A simple first approach is to create a frequency table for the unordered pair (i,j) where i and j are distinct airport codes.

We analyse the data of year 2008, and we get the route with most traffic as:

(LAX, SFO) 13390

(SFO, LAX) 13788

The busiest route is between LAX and SFO. We think it's because it's two of the biggest city in USA, there are many passengers fly between the two cities for business.
