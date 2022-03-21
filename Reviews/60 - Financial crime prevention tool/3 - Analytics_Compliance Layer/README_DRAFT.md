Grant Proposal | [60 - Financial crime prevention and risk intelligence on-chain tool](https://portal.devxdao.com/public-proposals/60)
------------ | -------------
Milestone | 3
Milestone Title | Analytics/Compliance Layer
OP | Aberto
Reviewer | Mickael Growet



# Milestone Details

## Details & Acceptance Criteria

**Details of what will be delivered in milestone:**

- Data analytics and compliance layer with functionalities to spot potential fraud and/or financial crime

**Acceptance criteria:**

- Incorporate an open source data visualization interface (eg. Kibana, Graphana) and provide a couple of analytics charts


**Additional notes regarding submission from OP:**
There was a major code refactor on the ORI project:
1. The project now is broken into several modules
    - ori-api (containing the REST API and database models)
    - ori-chains (contiaining the crawlers for different chains)
    - ori-client (a simple java client to use ori-api)
    - ori-dashboard (the deliverable for this milestone)
    - ori-frontend (the deliverable for the previous milestone)
    - ori-risk-metric (the module responsible for the risk metric calculations)
    - ori-shared (an implementation of DTO objects)
2. We migrated from a no-sql persistency layer (Elastic Search) to Postgresql together with hibernate
3. All dockers, jars are built with ./mvwn package command
4. The front-end also has been modified to add the capability to choose the chain
5. An ETH crawler has also been added to the project

**Notes from previous review**

With respect to this milestone specifically, after compiling the project and crawling the blockchain to populate the database (note that this might take a while) it is possible to access a preloaded dashboard after running the appropriate docker file through localhost:8088 with login: admin and password:admin (or any other value specified to the docker container running the dashboard)
**REST API**
- Swagger-ui: http://localhost:8080/q/swagger-ui/
- Swagger file: http://localhost:8080/swagger

**GraphQL API**
- GraphQL-ui: http://localhost:8080/q/graphql-ui/
- Schema: http://localhost:8080/graphql/schema.graphql

## Milestone Submission

The following milestone assets/artifacts were submitted for review:

Repository | Revision Reviewed
------------ | -------------
https://github.com/syntifi/ori/tree/main/ori-dashboard | 118a181

# Install & Usage Testing Procedure and Findings

Following the instructions in the README file of the repositoty : https://github.com/syntifi/ori and the file provided as a guide for this milestone

https://github.com/syntifi/ori/blob/main/TLDR-CSPR.md, the reviewer was
able to :

0- Install required dependencies on an Ubuntu 20.04 cloud server:

```console
Java 11 or above
Maven (3.8.1 or above)
Docker
Docker compose (version 1.29.2 or above)
Node.js (>=14.0.0) 
```

1. Test and build the project on an Ubuntu 20.04 cloud server

```bash
~/reviews/ori$ sudo ./mvnw clean compile

```

[60-3 clean-compile](assets/build.md "build")



2. Build libs, jars and images

```bash
~/reviews/ori$ sudo ./mvnw package -DskipTests
```

Output: 

[60-3 package](assets/build_libs.md "build_libs")


3. Create a docker network ori

```bash
~/reviews/ori$ docker network create ori
```

4. Start postgresql and api modules (docker images)

```bash
~/reviews/ori$ docker-compose -f ./ori-api/src/main/docker/docker-compose-jvm.yml up -d
Creating docker_ori-postgresql_1 ... done
Creating docker_ori-api_1        ... done
```

the reviewer was able to verify that the 2 docker images were up :

```bash
docker-compose -f ori-api/src/main/docker/docker-compose-jvm.yml ps
         Name                        Command              State                    Ports
----------------------------------------------------------------------------------------------------------
docker_ori-api_1          /deployments/run-java.sh        Up      0.0.0.0:8080->8080/tcp,:::8080->8080/tcp
docker_ori-postgresql_1   docker-entrypoint.sh postgres   Up      0.0.0.0:5432->5432/tcp,:::5432->5432/tcp
```

5. Launch the crawler for CSPR chain

```bash
~/reviews/ori$ docker-compose -f ./ori-chains/ori-cspr/src/main/docker/docker-compose.yml up -d
Creating docker_ori-cspr_1 ... done
```

the reviewer was able to verify that the docker image was up :

 ```bash
 ~/reviews/ori$ docker-compose -f ori-chains/ori-cspr/src/main/docker/docker-compose.yml ps
      Name              Command        State   Ports
----------------------------------------------------
docker_ori-cspr_1   /cnb/process/web   Up
 ```
The reviewer was then able to check crawling is up and running using the following command line :

 ```bash
docker-compose -f ori-chains/ori-cspr/src/main/docker/docker-compose.yml logs -f
 ```
![crawling](assets/crawling.png "crawling CSPR")

The crawler took a long time (at least a day) to populate the database.
The crawler retrieves all the blocks, accounts and transactions of the CSPR chain from Genesis (via RPC calls). 
The OP also offered the alternative to use an image with a database containing only recent blocks.
the reviewer wanted to make sure that the crawler completed the whole process of populating the data without errors.

6. Access the front-end

The reviewer was able to check that the front end webapp is up and running.
He could then access it using the url : http://server_ip_address:8080
![front_end](assets/front_end.png "front_end")

7. Check the database was populated 

The reviewer used  pgAdmin program to check the datas was successfully injected by the crawling process :
He verified 3 main tables :
   
   * Account :

   ![account](assets/account.png "account") 

   * Block:

   ![block](assets/block.png "block")
   
   * Transaction:

   ![transaction](assets/transaction.png "transaction")
   


9. Start the dashboard

```bash
~/reviews/ori$ docker-compose -f ori-dashboard/docker/docker-compose.yml up -d
Creating docker_dashboard_1 ... done
```

The reviewer was then able to check the dashboard is up and running using the following command line :

```bash
~/reviews/ori$ docker-compose -f ori-dashboard/docker/docker-compose.yml ps
       Name                   Command               State                        Ports
--------------------------------------------------------------------------------------------------------
docker_dashboard_1   sh ./docker/entrypoint.sh   Up (healthy)   0.0.0.0:8088->8088/tcp,:::8088->8088/tcp
```




This review focuses on the dashboard part.
The dashboard module uses the Apache Superset which is an open  data visualization and  exploration platform.
It is used here to have data analysis on transactions on the Casper blockchain. These analysis can be used for
coin tracing for example (used in AML).
the dashboaad is currently limited to transactions analysis but can be extended to explore other datas  such as blocks, validators etc ...
In this review,  we will carry out test scenarios on the following dashboard features :

- Transactions Monitor
- Charts 

1. Access the dashboard

The reviewer was able to connect to teh dashboard using  credentials provided in the Additional notes above.

![login](assets/dashboard/login.png "login")


2. Transactions Monitor
 
The reviewer was then able to load the transactions monitor  

![login](assets/dashboard/transaction_monitor_1.png "login")


The page displays a chart representing the coin flows (limited to 100 rows) and  informations like :

- Top 10 outflows
- Top 10 inflows
- Total amount
- Total transactions

### Filtering

The reviewer was able to filter datas using :

**'From Account' filter :**

From Account = account-hash-8c15bba2d147859c7b7a8f43028eeb4d3c9571c6e36dfecc97c77463d3af08cd

The dashboard was successfully loaded with all the transactions having this account as source

![filter1](assets/dashboard/transaction_monitor_2.png "filter1")


**'To Account' filter :**

From Account =  account-hash-00005f6d67c5303c206f5e678f4eb52703dc0665a7d08a12c8c6b55867da654d

The dashboard was successfully loaded with all the transactions having this account as recipient

![filter2](assets/dashboard/transaction_monitor_3.png "filter2")


### Adding filters

The dashboard offers teh possibility to add to the fields used to filter transaction datas.
The reviewer was able to add a new filter on the amount of transaction. 

![filter3](assets/dashboard/transaction_monitor_4.png "filter3")

Then he applied this filter with the value = 11 500 000 000 (motes). The datas was successfully loaded.

![filter4](assets/dashboard/transaction_monitor_5.png "filter4")


3- Charts

3.1 Displaying and updating charts

The dashboard offers the possibility of adding charts to visualize  transaction datas.

There are already 5 configured charts (embedded in the Transaction Monitor): 

![charts1](assets/dashboard/charts_1.png "charts1")

The reviewer was able to load all of these charts and apply changes to them.

* Example with coin flow chart : 

![charts2](assets/dashboard/charts_2.png "charts2")

The reviewer changed the 'Metric' filter from 'Count' to 'MAX':

![charts3](assets/dashboard/charts_3.png "charts3")

The  coin flow chart was updated accodingly. 

![charts4](assets/dashboard/charts_4.png "charts4")

The reviewer increased the 'Row limit' filter from 100  10000 rows (transaction hashes) and was able to update the chart:

![charts5](assets/dashboard/charts_5.png "charts5")

3.2 Adding new charts

The tool offers the possibility of adding many types of charts to explore transaction datas.

![charts6](assets/dashboard/charts_6.png "charts6")

The reviewer was able to add a new 'Pie' chart reflecting the count of transaction having a given amount.

![charts7](assets/dashboard/charts_7.png "charts7")



## Overall Impression of usage testing
TODO

Requirement | Finding
------------ | -------------
Project builds and runs without errors | PASS
Documentation provides sufficient installation/execution instructions | PASS
Project functionality meets/exceeds acceptance criteria and operates without error | PASS 


# Unit / Automated Testing
This part of the project uses Apache superSet to perform data analysis and exploration. 
There are currently no tools to perform automated tests on this platform. the reviewer was limited running the above test scenarios.

Based on last review , the project contains 24 unit tests accross all its modules . 
The tests can be executed from the console and cover both positive and negative test paths.

Requirement | Finding
------------ | -------------
Unit Tests - At least one positive path test | PASS
Unit Tests - At least one negative path test | PASS
Unit Tests - Additional path tests | PASS

# Documentation

### Code Documentation

TODO

The code in the project is essentially comment/documentation free, other than in the AMLRules.java class

Requirement | Finding
------------ | -------------
Low level function documentation | FAIL

### Project Documentation

TODO

As identified above, the project does NOT contain sufficient documentation describing the prerequisites and steps to set up, compile and run the project

Requirement | Finding
------------ | -------------
Sufficient Project Documentation | FAIL

### API Documentation

TODO

While some auto-generated API documentation is included, as identified above the lack of endpoint usage descriptions and/or input parameter format examples
has resulted in this Reviewer being unable to test certain API endpoint. As such, this requirement is not met.

Requirement | Finding
------------ | -------------
API documentation | FAIL

## Overall Conclusion on Documentation

TODO

Given the number of FAILs above, the overall conclusion has to be that this project falls short of the expected level of documentation.

# Open Source Practices

## Licenses

The Project is released under the MIT License

Requirement | Finding
------------ | -------------
OSI-approved open source software license | PASS

## Contribution Policies

The project contains a CONTRIBUTING policy that links to a Code of Conduct policy. Pull requests and Issues are enabled.

Requirement | Finding
------------ | -------------
OSS contribution best practices | PASS

# Coding Standards

## General Observations

TODO

Code is generally well-structured and very readable. The project as committed to GitHub cannot be run without finding the hidden workaround
of installing the Casper SDK Jar from file, nor can it properly be tested without spending hours of trial-and-error to infer
proper testing routes, parameter formats, etc. 

# Final Conclusion

TODO

The project provides the functionality described in the grant application and milestone acceptance criteria. 

The deliverable falls short in the following areas:
* Documentation

For a project of this magnitude, in both scope and cost, strict adherence to the minimal standards should be enforced, and therefore, it is this Reviewer's 
opinion that this milestone submission should fail, until the identified shortcomings are addressed. I do believe these are easily addressed, and I look forward
to reviewing the milestone again in the near future to re-assess my recommendation. Specifically, I believe the following items are critical to pass this review:

- Update the project documentation to ensure that someone who installs the listed pre-requisites and subsequently follows the outlined steps, ends up
with a working project 100% of the time.
- Update the project documenation to make clear how to use the project (e.g. upon completion of set-up, run initial synchronization before using the APIs)
- Update the API documentation with a description of the functionality of each endpoint, and an example value for each parameter from which the format can be inferred
- Improve the inline code documentation

# Recommendation

Recommendation | FAIL (request for improvements)
------------ | -------------

