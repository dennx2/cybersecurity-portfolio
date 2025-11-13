# Elastic Stack (ELK)

## Core Components

![elk-components](./screenshots/elk-components.png)

**Beats**: collect data from multiple agents.

- Each beat is a single-purpose agent that sends specific data to Elasticsearch.

**Logstash**: collects data from beats, ports, or files, parses/normalizes it into field value pairs, and stores them into Elasticsearch. A Logstash configuration file is divided into three parts:

- **Input**: where the user defines the source from which the data is being ingested.
- **Filter**: where the user specifies the filter options to normalize the log ingested above.
- **Output**: where the user wants the filtered data to be sent. It can be a listening port, Kibana Interface, Elasticsearch database, or file.

**Elasticsearch**: acts as a database used to search and analyze data.

- A full-text search and analytics engine for JSON-formatted documents.
- Stores, analyzes, and correlates data and supports a RESTful API for interacting with it.

**Kibana**: responsible for displaying and visualizing the data stored in Elasticsearch.

- Web-based data visualization tool that works with Elasticsearch.

## KQL (Kibana Query Language)

**Free text Search**: allows users to search for logs based on text only.

- Whole term search, e.g. `"United States"`
- With wildcard, e.g. `"United*"`
- With Logical Operators (AND | OR | NOT), e.g. `"United States" AND "Virginia"`, `"United States" AND NOT ("Florida")`

**Field-based search**: use `Field: Value` to search for logs

- e.g. `Source_ip : 238.163.231.224 AND UserName : Suleman`

## Practice Questions

### Discovery Tab

**Q1.** Which user is responsible for the overall maximum traffic?
   ![elk-discovery-q1](./screenshots/elk-discovery-q1.png)
   Answer: James

**Q2.** Apply Filter on UserName Emanda; which SourceIP has max hits?
   ![elk-discovery-q2](./screenshots/elk-discovery-q2.png)
   Answer: 107.14.1.247

**Q3.** On 11th Jan, which IP caused the spike observed in the time chart?
   ![elk-discovery-q3](./screenshots/elk-discovery-q3.png)
   Answer: 172.201.60.191

**Q4.** How many connections were observed from IP 238.163.231.224, excluding the New York state?
   ![elk-discovery-q4-1](./screenshots/elk-discovery-q4-1.png)
   ![elk-discovery-q4-2](./screenshots/elk-discovery-q4-2.png)
   ![elk-discovery-q4-3](./screenshots/elk-discovery-q4-3.png)
   Answer: 48

**Q5.** Create a table with the fields IP, UserName, Source_Country and save.
   ![elk-discovery-q5](./screenshots/elk-discovery-q5.png)

### KQL

**Q1.** Create a search query to filter the logs where Source_Country is the United States and show logs from User James or Albert. How many records were returned?
![elk-kql-q1](./screenshots/elk-kql-q1.png)

```kql
Source_Country: "United States" and UserName : "James" or UserName :"Albert"
```
Answer: 161


**Q2.** A user Johny Brown was terminated on the 1st of January, 2022. Create a search query to determine how many times a VPN connection was observed after his termination.
   ![elk-kql-q2](./screenshots/elk-kql-q2.png)
   Answer: 1

### Creating Visualization

Create a table to show the user and the IP address involved in failed attempts.

![elk-viz-q0-1](./screenshots/elk-viz-q0-1.png)
![elk-viz-q0-2](./screenshots/elk-viz-q0-2.png)

**Q1.** Which user was observed with the greatest number of failed attempts?
   ![elk-viz-q1-1](./screenshots/elk-viz-q1-1.png)
   ![elk-viz-q1-2](./screenshots/elk-viz-q1-2.png)
   ![elk-viz-q1-3](./screenshots/elk-viz-q1-3.png)
   Answer: Simon

**Q2.** How many wrong VPN connection attempts were observed in January?
   ![elk-viz-q2](./screenshots/elk-viz-q2.png)
   Answer: 274
