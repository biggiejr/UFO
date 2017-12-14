# Performance issues with growing dataset in Neo4j
## Abstract
Neo4j is a good database for data with multiple relationships but it has its limitations. 
When the data in a neo4j database starts to increase it becomes slow in many different aspects. This can result to very slow response time and bad user experience. There are some steps that can remove many of the shortcomings.

## Our experience with Neo4j 
Neo4j is a Graph database that enables multiple relationships between nodes in an abstract way that allows for a flexible development.
Using Neo4j in a project with a lot of relationships sounded like the best option. Everything was smooth in the beginning and the Neo4j was fast and smooth, it gave as what we needed and it seemed that we made the best possible choice to our knowledge. There were recursive comments, a mapping on who wrote what and who commended on what. It allowed us to display data in a graceful way without lots of complicated queries.
However, after some time that the data started expanding some of the problems started to surface. 	
Investigating the performance issues

When the dataset started growing, the cypher queries started to become slower and unresponsive. We started suspecting the open connections on the system for the hang time we were getting so we starting investigating that but there is no easy way to check the amount of open connection on Neo4j. 
	After a lot of search on the web we implemented the best practices that we found on closing sessions and connection on our backend but nothing seemed to have changed. The database was still getting slower and slow. After further investigation we found that our main cause of performance problems lays in  Neo4j’s orderby method. As it turned out after research, Neo4j does not take indexes into consideration while ordering, so our index on created_at for example was, in this case, useless ( figure 1 & 2 ). Consequences of this imperfection in the system resulted in long waiting time for simple actions. 

![alt text](https://github.com/biggiejr/UFO/blob/master/images/1.png)
### Figure 1 : Fetch 20 stories ordered by creation day (60183 ms.)
 

![alt text](https://github.com/biggiejr/UFO/blob/master/images/2.png)
### Figure 2 : Fetch 20 stories (1168 ms.)
	

Initially we did not thought to check the cypher because we timed the queries to begin with and the response time was always a two digit milliseconds value which we have seen as sufficient so naturally the problem was thought to be something else. 
	After further research it was found out that there is a limitation to the connections that the database can have open at a time. Also since Neo4j is java based it runs on a JVM (java virtual machine) it can become very heavy and slow if there are many connections simultaneously. Investigating the documentation the HA (high availability ) (Figure 3) setup came across, it runs on the principle of multiple Neo4j instances running at the same time while all of them point to the same data. That setup allows the system to use more connections at a time. The downside of this approach is, that all the neo4j instances have to run on the same machine making it an expensive solution since neo4j uses higher amount of RAM for a single instance, compared to the relational databases.  
	
However, if a user decides to use a community edition of the Neo4j, same as we did, he will lack some essential tools needed to transfer data, thus, making this High-availability  solution not relevant since one is not able to do it without jeopardizing data.



![alt text](https://github.com/biggiejr/UFO/blob/master/images/3.png)
### Figure 3: Neo4j HA setup



Solution
	After experimenting with a wide range of Cyphers, an unorthodox solution was attempted. The idea was to run multiple queries that would supplement a single heavy  Cypher which at the time would complete at around 3 minutes ( figure 4 & 5). This approach establishes way more connections than the first one but keep the database busy less and therefore the connections would be released faster for new connections to take place.
 

![alt text](https://github.com/biggiejr/UFO/blob/master/images/4.png)
### Figure 4 : Old Cypher 1

![alt text](https://github.com/biggiejr/UFO/blob/master/images/5.png)
### Figure 5 : Old Cypher 2

The cyphers to replace the old heavier single cypher would be one to get 20 stories and right after 20 queries to find how many comments exist under each story individually.
	This approach letter was iterated through once again to use the same session for all 21 cypher queries and then close it once all all of them were finished ( Figure 6 ). 

![alt text](https://github.com/biggiejr/UFO/blob/master/images/6.png)
### Figure 6 : Multiple lightweight Cyphers implementation.

### Future plans 
Eventually when the orderby bug is fixed by Neo4j, we would like to stick to our initial idea with only one query. However, even though it is anounced to be fixed in version 3.4, the release date remains, unfortunately, still unknown.

### References: 
https://stackoverflow.com/questions/23835349/does-cyphers-order-by-uses-the-index
https://github.com/neo4j/neo4j/issues/6584
https://neo4j.com/developer/guide-performance-tuning/
https://stackoverflow.com/questions/24646962/neo4j-how-to-setup-failover-in-community-edition

 #### Authors: Athinodoros Sgouromallis & Martin Macej

