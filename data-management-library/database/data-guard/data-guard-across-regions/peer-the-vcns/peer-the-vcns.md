# Connect the VCNs by Peering

## Introduction
In order to configure Data Guard across regions we must first set up remote VCN peering.  Remote VCN peering connects VCNs in different regions together.  The peering allows resources such as the database with private IP address to communicate across regions.
A picture of the remote VCN peering is shown below.  

![This image shows the basic layout of two VCNs that are remotely peered, each with a remote peering connection on the DRG](./images/network_remote_peering_basic.png)

If you don't have admin policies you will need to add remote peering policies.  A diagram of the policies follows.  Refer to the documentation for more information.

![This image shows the two policies for VCNs in different regions but in the same tenancy.](./images/network_remote_peering_policy_same_tenancy.png)

The DRG or Dynamic Routing Gateway must be set up on both VCNs and the RPC or Remote Peering Connection is then configured to connect the two VCNs.  

The route rule and security list must also be configured to access the resources in the VCN.

Estimated lab time: 10 minutes

### Objective
- Peer the VCNs together

### Prerequisite

- User policies to set up the VCN peering  (if you do not have admin policies, refer to the documentation to add peering policies)

- VCNs already created in your primary and standby regions

- Oracle Database Cloud Service created in your primary region


## **STEP 1:** Create the Dynamic Routing Gateways
1. Create the DRGs in both VCNs.  

2. Select a region to start with.  You can pick either the primary side or the standby.

3. Select the menu Networking

4. Ensure you are in the right region and compartment

5. Under Customer Connectivity select Dynamic Routing Gateway

6. Click Create Dynamic Routing Gateway

7. Name your DRG and click create


  ![drg-create](./images/drg-create-copy.png)


## **STEP 2:** After creating the DRG, you must attach it to your VCN

1. Navigate to your VCN details, select Dynamic Routing Gateways, and click Attach Dynamic Routing Gateways

2. Select your compartment and DRG

  ![attach-drg](./images/attach-drg-copy.png)

## **STEP 3:** Do the same steps above in your second region


## **STEP 4:** Now let's create the Remote Peering Connections in both VCNs

1. Create the Remote Peering Connections in both VCNs.

2. Select the DRG you created.

3. Ensure you are in the correct region and compartment.

4. Select from Resources the Remote Peering Connections.

5. Click Create Remote Peering Connection.

6. Provide a name for the connection.

7. Create the RPC.



  ![image-20210124111112366](./images/image-20210124111112366.png)



## **STEP 5:** Now do the same on the other DRG in the other region

## **STEP 6:** Establish connection
Once you have both DRGs and RPCs created.  You must decide which side accepts a connection, and which side requests a connection.

For our lab, we'll use the standby side as the acceptor, and the primary side as the requestor.  

1. Navigate to the standby region.

2. Record the OCID of the RPC on the standby side.  You will provide this to the primary side later to establish the peering.

  ![image-20210124114804001](./images/image-20210124114804001.png)



3. Navigate to your primary region.

4. Select the DRG then the RPC you created.

5. On the right 3 dot action menu, select Establish Connection.



  ![image-20210124114006763](./images/image-20210124114006763.png)

6. Enter the standby region you will establish the connection with.

7. Paste in the OCID of the RPC from the standby region.

8. Click Establish Connection.  Your connection will be established in a few minutes.



  ![image-20210124114207345](./images/image-20210124114207345.png)



The Peering Status will show Peered if it is successful.

You may now [proceed to the next lab](#next).

## Acknowledgements
* **Author** - Milton Wan, Database Product Management, Dec 2020  
* **Last Updated By/Date** - Milton Wan, Jun 2021
