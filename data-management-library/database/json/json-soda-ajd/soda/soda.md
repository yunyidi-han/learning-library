# Using SODA for REST

## Introduction

So far, all collection operations have been issued from a UI in the browser. In a real application these operations would be called directly from a programming language (for example using the open-source SODA drivers for Java, Python or NodeJS). Another option is to use REST.

Estimated Lab Time: 15 minutes

### Objectives

In this lab, you will:

* Perform Simple REST Operations in Oracle Cloud Shell
* Insert Data into the Collection

### Prerequisites

* Have successfully created a JSON collection in Autonomous JSON Database and have inserted few documents

## **STEP 1:** Perform Simple REST Operations in Oracle Cloud Shell

1. A simple REST request can be done from the browser - open a new window and copy the URL from the 'JSON Workshop' or the 'SQL Developer Web' into it.

	The URL should look similar to this:

	```
	https://ppkenzghg74avsq-atp19cdb.adb.eu-frankfurt-1.oraclecloudapps.com/ords/admin/_sdw/?nav=worksheet
	```

	*Note* - In this lab, we refer *your URL* in code snippets where your URL is similar to `ppkenzghg74avsq-atp19cdb.adb.eu-frankfurt-1.oraclecloudapps.com`. Replace *your URL* with your URL in all the code snippets wherever mentioned.

2. In the URL, remove the part - *_sdw/?nav=worksheet* and replace it with *soda/latest*.

	```
	https://<your URL>/ords/admin/soda/latest/
	```

	Your URL should now look like this:

	```
	https://ppkhnzjhg74axsq-bedasatpdb.adb.eu-frankfurt-1.oraclecloudapps.com/ords/admin/soda/latest
	```

3. Hit Enter to load this URL. You should see a JSON document which lists s - it shows the 'products' collection with some additional information.

	![](./images/additional-info.png)

4. In order to see the contents of the collection (the documents) all we have to do is append */products* (the collection name) to the URL and hit Enter.

	```
	https://<your URL>/ords/admin/soda/latest/products
	```

	Your URL should now look like this:

	```
	https://ppkhnzjhg74axsq-atp19cdb.adb.eu-frankfurt-1.oraclecloudapps.com/ords/admin/soda/latest/products
	```
	![](./images/documents-1.png)

5. The browser is limited to GET requests. For further operations we need to also perform other requests. For this we switch to use the 'curl' command in the Oracle Cloud Shell. If you are familiar with other REST tools like Postman you can also use them for the following examples.

	Navigate to Oracle Cloud Console and click on Cloud Shell icon.

	![](./images/ocshell.png)

6.	In the cloud shell, use the same URL to make a GET request as follows:

	```
	curl -X GET https://<your URL>/ords/admin/soda/latest
	```

	Your URL should now look like this:

	```
	curl -X GET https://ppkhnzjhg74axsq-bedasatpdb.adb.eu-frankfurt-1.oraclecloudapps.com/ords/admin/soda/latest
	```

	You'll see an authorization error. Oracle's security mechanisms kicked in as this REST request came from outside the database cloud service. Instead of explaining different authentication mechanisms here we turn it off. You would not do that in a real production system!

	![](./images/error.png)

7. Navigate to the JSON workshop tab, click on the navigation menu on the top left and select **SQL** under Development.

	![](./images/nav-sql.png)

8. Copy and paste the procedure below in SQL Developer Web worksheet and run it.

	```
	<copy>
	BEGIN
		ORDS.delete_privilege_mapping('oracle.soda.privilege.developer', '/soda/*');
		COMMIT;
	END;
	/
	</copy>
	```

	![](./images/remove-error.png)

8. Navigate back to the tab with Oracle Cloud Shell. Running the same curl command again should now return the same result that we have seen in the web browser : the contents of the 'products' collection.

	```
	curl -X GET https://<your URL>/ords/admin/soda/latest
	```

	Your URL should now look like this:

	```
	curl -X GET https://ppkhnzjhg74axsq-bedasatpdb.adb.eu-frankfurt-1.oraclecloudapps.com/ords/admin/soda/latest
	```
	![](./images/success.png)

9. We can also run a QBE using curl. This would be a post request. Make sure you add *?action=query* to the URL.

	The following example issues a QBE selecting all products costing more than 5.

	```
	curl -X POST -H "Content-Type: application/json" --data '{"price":{"$gt":5}}' https://<your URL>/ords/admin/soda/latest/products?action=query
	```

	Your URL should now look like this:

	```
	curl -X POST -H "Content-Type: application/json" --data '{"price":{"$gt":5}}' https://ppkhnzjhg74axsq-atp19cdb.adb.eu-frankfurt-1.oraclecloudapps.com/ords/admin/soda/latest/products?action=query
	```

	![](./images/more5.png)

10. We can insert a new document, also using a POST request but without the ?action=query` at the end of the URL.

	```
	curl -X POST -H "Content-Type: application/json" --data '{"id": 1414,"type": "Toy","title": "E.T. the Extra-Terrestrial","condition": "washed","price": 50.00,"description": "50cm tall plastic figure"}' https://<your URL>/ords/admin/soda/latest/products
	```

	Your URL should now look like this:

	```
	curl -X POST -H "Content-Type: application/json" --data '{"id": 1414,"type": "Toy","title": "E.T. the Extra-Terrestrial","condition": "washed","price": 50.00,"description": "50cm tall plastic figure"}' https://ppkhnzjhg74axsq-atp19cdb.adb.eu-frankfurt-1.oraclecloudapps.com/ords/admin/soda/latest/products
	```

	![](./images/created.png)

11. To verify that the new document was inserted, navigate to the tab with SQL Developer Web, click on the navigation menu on the top left and select **JSON** under Development.

	![](./images/nav-json.png)

	If you use the following QBE in the JSON Workshop by copying and pasting the following query in the worksheet and running it, you should see a new document.

	```
	<copy>
	{"id":1414}
	</copy>
	```

	![](./images/proof.png)	

## **STEP 2:** Insert Data into the Collection

1. Now, let's do a bulk load to insert more data. Navigate back to the tab with Oracle Cloud Shell.

	![](./images/soda2-1.png " ")

2.	The test data has been uploaded to the object store. Run the following pre-authenticated link that allows us to retrieve the data.

	```
	<copy>
	curl https://objectstorage.us-ashburn-1.oraclecloud.com/p/D2qfIeCdhjmT3tik7QZQgVIO1bcV3jbW5vgqKMzWM3_GtLCOgZvDvz9JhaPqjhCB/n/c4u03/b/data-management-library-files/o/testdata.json --output testdata.json
	</copy>
	```

	![](./images/soda2-2.png " ")

3.	Confirm the download by doing a *ls*. You should see the file testdata.json.

	![](./images/soda2-3.png " ")

4.	Now, we take this file (which is one large JSON array of objects) and bulk insert it to the collection using REST. Each object of the JSON array will be inserted into the products collection as a separate document.

5.	The following POST command with the action clause 'insert' will take the single JSON array and insert the embedded objects as separate documents. Replace *your URL* with your URL.

	```
	curl -X POST --data-binary @testdata.json  -H "Content-Type: application/json" https://<your URL>/ords/admin/soda/latest/products?action=insert
	```

	Your URL should now look like this:

	```
	curl -X POST --data-binary @testdata.json  -H "Content-Type: application/json" https://ppkhnzjhg74axsq-atp19cdb.adb.eu-frankfurt-1.oraclecloudapps.com/ords/admin/soda/latest/products?action=insert
	```

	![](./images/soda2-4.png " ")

6. Navigate back to the tab with JSON workshop to run this in the worksheet to verify that you have many more movies in the collection.

	```
	<copy>
	{}
	</copy>
	```
	![](./images/soda2-5.png " ")

You may now [proceed to the next lab](#next).

## Learn More

* [Overview of SODA](https://docs.oracle.com/en/database/oracle/simple-oracle-document-access/adsdi/overview-soda.html#GUID-BE42F8D3-B86B-43B4-B2A3-5760A4DF79FB)
* [SODA for REST Overview](https://docs.oracle.com/en/database/oracle/simple-oracle-document-access/rest/adrst/rest.html#GUID-CDCB8C6D-E830-4851-AED0-1BCBBCD1AC9A)
* [Using SODA for REST](https://docs.oracle.com/en/database/oracle/simple-oracle-document-access/rest/adrst/using-soda-rest.html#GUID-C67498D7-5783-4969-80EF-C180CEC1144A)
* [Use SODA for REST with Autonomous Database](https://docs.oracle.com/en/cloud/paas/autonomous-json-database/ajdug/ords-develop-oracle-soda-data-services.html)
* [Overview of SODA Filter Specifications](https://docs.oracle.com/en/database/oracle/simple-oracle-document-access/adsdi/overview-soda-filter-specifications-qbes.html#GUID-CB09C4E3-BBB1-40DC-88A8-8417821B0FBE)

## Acknowledgements

- **Author** - Beda Hammerschmidt, Architect
- **Contributors** - Anoosha Pilli, Product Manager, Oracle Database
- **Last Updated By/Date** - Anoosha Pilli, Brianna Ambler June 2021
