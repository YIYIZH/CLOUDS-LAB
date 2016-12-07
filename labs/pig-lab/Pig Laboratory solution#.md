# Pig Laboratory solution#

##Exercise 1:: Word Count
### Questions:
- **Q1: Compare execution times between your Pig and Hadoop MapReduce jobs: which is faster? What are the overheads in Pig?**
  
	Hadoop Mapreduce has better performance but require more develop time. Pig is short and quicky written, optimizations are already done automatically. But it supports limited set of operations, while hadoop mapreduce can be controlled and designed be developpers. 

- **Q2: What does a GROUP BY command do? In which phase of MapReduce is GROUP BY performed in this exercise and in general?**

	Group by: Group rows by one or more keys.   
	In map phase it sets the key for group, here the key is each word. The Reducer phase processes collection of values for each key and write them to the disk. Here it does the grouping and sorting by word.
- **Q3: What does a FOREACH command do? In which phase of MapReduce is FOREACH performed in this exercise and in general?**

	FOREACH: iterates row-by-row through the whole dataset to performs operations.   
	In reduce phase FOREACH is used. It is used to count the frequncy (sum the values) for each word(key).
	
##Exercise 2:: Working with Online Social Networks data

##Exercise 3:: Find the number of two-hop paths in the Twitter network
### Questions:
- **Q1: What is the size of the input data? How long does it take for the job to complete in your case? What are the cause of a poor performance of a job?**   
	- Size: 11.7 Mb, 904215 records
	- Big? no
	- How long: Pig script completed in 29 minutes, 36 seconds and 405 milliseconds (1776405 ms)
	- Causes:   
Cluster load   
I/O (multiple jobs)   
Join operation requires shuffling data alot
	
	
- **Q2: Try to set the parallelism with different number of reducers. What can you say about the load balancing between reducers?**   

	Some reducers have to work as 2-3 times as the other reducers (not good).


- **Q3: Have you verified your results? Does your result contain duplicate tuples? Do you have loops (tuples that points from one user to the same user)? What operations do you use to remove duplicates?**

	There are no duplicate tuples and no loops. I use the DISTINCT operation.
- **Q4: How many MapReduce jobs does your Pig script generate? Explain why**   

	- There are 4 jobs
		- Load dataset A
		- Load dataset B
		- Join operation
		- Distinct operation
	- Verify:
		- EXPLAIN result; #each MRNode is one MR job
		- ```pig -x local -e 'explain -script /home/ntkhoa/IdeaProjects/CloudLabEurecom-Pig/scripts/ex2/tw-join.pig' > ~/explain.txt grep MapReduce ~/explain.txt```


##Use-case:: Working with Network Traffic Data

##Use-case: Working with an Airline dataset

