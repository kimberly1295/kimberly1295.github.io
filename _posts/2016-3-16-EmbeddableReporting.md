---
layout: post
title: Embeddable Reporting
permalink: /embeddable-reporting/
---

##Embeddable Reporting
Embeddable Reporting lets you run IBM Cognos Business Intelligence reports within your Bluemix environment. The reporting service is based on Cognos Business Intelligence version 10.2.2. Embeddable Reporting allows reports to be included in the application. 

In this tutorial, 3 services will be use to bind to the application:

**Data Management Services** (use to store report artifacts)
 - IBM Cloudant NoSQL DB
 
 **Database Service** (use to store report data)
 - IBM dashDB
 - 
**Reporting Service** (use to create reports)
 - Embeddable Reporting

>**Copy a Github Repository**

>Having a basic background in web application development is required in this tutorial.

<br>

####Copy a Github Repository
1. Open a web browser and login to [Github](https://github.com) .
1. Go to the Github Repository https://github.com/kimberly1295/EmbeddableReporting
1. Fork the repository by clicking the Fork button to get a copy to your account.

<br>

####**Create a Bluemix Devops Project based on the Github Repository**

1. Using the same browser, open a new tab and login to [Bluemix Devops](https://hub.jazz.net/). 

2. Click the `CREATE PROJECT` button.

3. Name your project `EmbeddableReporting`.

4.  Click `Link to an existing Github repository`.

5. Select the repository `<username>/EmbeddableReporting` from the dropdown list.

6. On the `Select project contents` section, make sure that the following options are selected:

	||||
	|---|---|---|
		|**Private Project**| checked |
		| **Add features for Scrum development** | checked |
		| **Make this a Bluemix Project** | checked |
		| **Region** | IBM Bluemix US South |
		| **Organization** | you may leave the default selection |		
		| **Space** | dev |

7. Click `CREATE`.

<br>

####**Create a Build Stage**

1. Click the `BUILD & DEPLOY` button.

2. Click the `ADD STAGE` button. Change the stage name `MyStage` to `Build Stage`.

3. On the `INPUT` tab, set the following values:

	||||
	|---|---|---|
		| **Input Type** | SCM Repository |
		| **Git URL** | https://github.com/kimberly1295/EmbeddableReporting.git |
		| **Branch** | master |
		| **Stage Trigger** | Run jobs whenever a change is pushed to Git |

4. On the `JOBS` tab, click the `ADD JOB` button and select `BUILD` as the Job Type.
5. Change the job name `Build` to `Gradle Assemble` and set the following values:

	||||
	|---|---|---|
		| **Builder Type** | Gradle |		
		| **Build Shell Command** | `#!/bin/bash`<br>`export PATH="$GRADLE2_HOME/bin:$PATH"`<br>`gradle assemble`  |	
		| **Stop running this stage if this job fails** | checked |

6. Click the `SAVE` button.

<br>

####**Create a Deploy Stage**

1. Click the `ADD STAGE` button to create a deploy stage. Change the stage name `MyStage` to `Deploy Stage`.

2.  On the `Input` tab, set the following values:

	||||
	|---|---|---|
		| **Input Type** | Build Artifacts |
		| **Stage** | Build Stage |
		| **Job** | Gradle Assemble |
		| **Stage Trigger** | Run jobs when the previous stage is completed |

3.  On the `JOBS` tab, click the `ADD JOB` button and select `DEPLOY` as the Job Type.

4.  Change the job name `Deploy` to `CF Push to Dev Space` and set the following values:

	||||
	|---|---|---|
		| **Deployer Type** | Cloud Foundry |		
		| **Target** | IBM Bluemix US South - https://api.ng.bluemix.net |		
		| **Organization** | you may leave the default selection |		
		| **Space** | dev |	
		| **Application Name** | blank |		
		| **Deploy Script** | `#!/bin/bash`<br>`cf push ers-<your_name> -m 512M -p build/libs/EmbeddableReporting.war`  |	
		| **Stop running this stage if this job fails** | checked |

5.  Click `SAVE`.

<br>

####**Deploy the Application using Delivery Pipeline**

1. On the `BUILD & DEPLOY` tab, click the `Run Stage` icon of the `Build Stage`.

	>Make sure all the stages are passed before you proceed to the next step.

2. Go to [IBM Bluemix](ibm.biz/bluemixph) Website and login.

3. Click the widget of your application `ers-<your_name>`.

4. Click the `ADD A SERVICE OR API` to add the services to your application.

5. In the `CATALOG` page, add the following services:
	||||
|---|---|---|
		| **Data management service** | IBM Cloudant NoSQL DB |		
		| **Database service** | IBM dashDB |		
		| **Business Analytics** | Embeddable Reporting |	

7. Click `CREATE`. When asked to restage your application, click the `RESTAGE` button. Wait for your application to restage.

8. Open a new tab and go to `http://ers-<your_name>.mybluemix.net` to test if the application can connect to the Embeddable Reporting service. 

9. Click the `Upload` button under  the `Add Data to Cloudant` header.
10. Add the incidents.json file
 >The Upload button adds a NoSQL JSON style format bulk insertion to Cloudant. 
 >Bulk insertion add multiple entries to the Cloudant.

11. To view the inserted entries, go back to [IBM Bluemix Dashboard](https://ibm.biz/bluemixph).

12. Under `Services`, click the created `Cloudant No SQL` service.

13. Click the `Launch Button` at the upper right corner. It will redirect you to your `Cloudant Web Console`. 

14. Using the  Cloudant Web Console, check if the database name `incidents` exist.

####**Creating a Warehouse to transform JSON data into relational tables**

15.   From the `Cloudant Dashboard`, navigate to the Warehousing tab, and click the `New Warehouse`. 

16.  Select the `incidents`  database. 

17. Click the `Warehousing` and Select `Create a dashDB warehouse`. 
>Note: Only one warehouse can be created.

17. Provide your  `username` and `password` credentials that you use to access your `Bluemix account`.

18. For the warehouse name, input  `incidents-warehouse`

19. Under the Data Sources, input `incidents`

18. Select the `dashDB service instance`, then select `dashDB-<name>`, the service you have binded earlier.

18. The newly created warehouse can be viewed in the dashDB dashboard.

<br>

####**Access Warehouse in the IBM dashDB instance**
1. Under `Services`, click the `dashDB` service.

2.  Select the `Go to your tables`, then at the `Table Name` select `incidents`

3. This displays the json file was transformed into a relational table.

####**Use Cloudant as connect source for Embeddable Reporting service**
1. From the `VCAP_SERVICES`, under the `CLOUDANTNOSQLDB`, copy the `url` value of the cloudant service.

2. Under `Services`, click the `Embeddable` service.

3. Under `Repository URI`, enter the `url` value of the cloudant service and click `start`

####**Add IBM dashDB Connection details in Emebbedable Reporting service instance as data source**
4. Select the `New Package`
 
5. From the `VCAP_SERVICES`, under the `DASHDB`, copy the `username`, `password`, `jdbcurl` values of the dashDB service.

5.  	Enter the following credentials:
||||
	|---|---|---|
		| **Name** | ERS |		
		| **Description** | you can leave this blank |		
		| **JDBC URI:** | `jdbcurl` |		
		| **User Name:** |  `username` |	
		| **User Password:** |`password` |		
Then select `create` 	
	
6. Under `Report Definitions` select the `New Report Definitions`

7. Select the `SQL Chart`

8. Select  `datasource`

8. Enter the following SQL Statement 
`select * from incidents`

9.  Click `validate`, if there are no errors, select `OK`

####**Creating of Reports using the Embeddable Reporting Service**
10.  Under columns, select the Clustered Column

11. On the left pane, select  `data items`, it is located between the source and the toolbox icons. Here are the Query Items generated from the relational database.

12.  Select `view` , select `Queries` then Double-click the `Query 1`

13. Select `toolbox` from the left pane, drag the `data item` to the Data Items

14. Change the name to  `count_incidents` and under the Expression Definition enter the following `count([SQL1].[IncidntNum])`. Click `validate`, the green icon with a check mark, once no errors displayed select OK.

15.  We will create another data item called `year` then repeat step number 4 for the creation of new data item.

16.  Change the name to  `year` and under the Expression Definition enter the following `substr([SQL1].[Date],1,4)`. Click `validate`, the green icon with a check mark, once no errors displayed select OK.

17.  Select `View`, then `Report Pages`. Select `Page1`from the list of Report Pages.

18. Under  `data items`, drag the following: 
||||
	|---|---|---|
		| **Y-axis or Values** | count_incidents |		
		| **x-axis or Categories** | Year |		
		| **Series** | PdDistrict |	
> The chart created displays from 2003 to 2015 of the number of incidents happen per year.

18. Click the run report, play icon, to display the chart.

19. Save the report by clicking `File` then `Save` on the toolbar. 

20. Reports can be authored in the Embeddable Service Dashboard.

22.  Access this for the authentication between the application and the service `https://erservice-impl.ng.bluemix.net/ers/swagger-ui/`

23. Select  `Connections` then `Post`, remove the values in body and under `Model Schema` select the paramater values.

24. Select the `Try it out!` button 

25. From the `VCAP_SERVICES`, under the `erservice`, copy the `userid` and `password` values of the erservice. 

25. Enter your  the following to the credentials:
|---|---|---|
		| **UserId** | `userid` |		
		| **Password** | `password` |		

23. An HTTP status code of 200 is returned if the credentials are entered correctly and response is returned with the values of `bundleUri` and `connectionId`
> {
  "bundleUri": "https://2549fade-55f6-4bfc-8791-f06f99e1feeb-bluemix:c288fcf5d6cfa5df0cc7f8480c2c00b298986c2b4567466ff699236d287403d7@2549fade-55f6-4bfc-8791-f06f99e1feeb-bluemix.cloudant.com",
  "connectionId": "MMamLqLiQxuHTLWDqcdHBA"
} 

22. Select  `Definitions` then `GET /definitions/`, Select the `Try it out!` button 

22. Copy the `url` value of the response 
> {
    "name": "chart 2 report",
    "url": "/ers/v1/definitions/1a6ea150a995dd0d87e44dd9bbca0d0d",
    "type": "application/vnd.ibm.ers.v1.definitions+json",
    "id": "1a6ea150a995dd0d87e44dd9bbca0d0d"
  },

22. Open a new tab and go to `http://ers-<your_name>.mybluemix.net`  to access your application. 

23. Under the `View Reports` label enter the `url` value from number 20, select view report button.

23. Select the link `access report` to view the chart created using the embeddable reporting service in the application. 

<br>

####**Delete the Application and Object Storage Service**
1. Go to [IBM Bluemix](ibm.biz/bluemixph) Website and click the `Dashboard`.
2. From the `Applications` section, click the `gear` icon in the widget of the `ers-<your_name>` application.
3. Select the `Delete App` from the list.
4. In the `Services` tab, select the `Cloudant`, `dashDB`, `Embeddable Reporting` service.
5. Make sure that the route is selected as well.
6. Click the `Delete` button.

####End of Tutorial
