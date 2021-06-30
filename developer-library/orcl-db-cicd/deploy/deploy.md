# Deploy Python App on Container Cluster (OKE)

## Introduction

At this point we have all the components we need to deploy our Python micro web service application to OKE cluster on Oracle Cloud. This requires a new pipeline definition in **wercker.yml** file. We will call it **deploy**.

Automated deployment uses YML templates received from the original [vltabacaru/orcl-ws-cicd] GitHub repository we forked.

Estimated Lab Time: 30 minutes

## **Step 1:** Add Pipeline to Wercker.YML

1. Add this section at the end of your wercker.yml file (leave a blank line between build section and deploy section):

    ````
    deploy:
        box: python:3.7
        steps:

        # Step 1: expand environment variables in .template files and remove extension
        - bash-template

        # Step 2: create a directory and move .yml template files inside
        - script:
            name: prepare Kubernetes files
            code: |
              mkdir $WERCKER_OUTPUT_DIR/kubernetes
              mv kubernetes_*.yml $WERCKER_OUTPUT_DIR/kubernetes
        # Step 3: create Kubernetes entities if missing
        - kubectl:
            name: deploy to kubernetes
            server: $OKE_MASTER
            token: $OKE_TOKEN
            insecure-skip-tls-verify: true
            command: apply -f $WERCKER_OUTPUT_DIR/kubernetes/

        # Step 4: wait 120 seconds for the deploy to be successful
        - kubectl:
            name: set deployment timeout
            server: $OKE_MASTER
            token: $OKE_TOKEN
            insecure-skip-tls-verify: true
            command: patch deployment/orcl-ws-app -p '{"spec":{"progressDeadlineSeconds":120}}'

        # Step 5: watch the deployment. If after 120s pods are considered healthy, the step will exit 0
        - kubectl:
            name: check deployment status
            server: $OKE_MASTER
            token: $OKE_TOKEN
            insecure-skip-tls-verify: true
            command: rollout status deployment/orcl-ws-app
    ````

2. This Pipeline has five Steps, and uses the two templates provided as YML files, one for the Docker container deployment, the other for the Docker service. It also uses some default Wercker variables, like WERCKER_OUTPUT_DIR, and the custom environment variables we defined, like OKE_MASTER. Last three steps use Kubectl tool to apply the definition in the YML files, wait for two minutes, and verify the deployment.

3. Commit and push changes to the master branch.

    ````
    git commit -a -m "Add build pipeline"
    git push
    ````

4. Now wercker.yml has two sections:

    ````
    build:
        box: python:3.7
        steps:
    ...

    deploy:
        box: python:3.7
        steps:
    ...
    ````

## **Step 2:** Deploy Pipeline

1. We need to add the second pipeline to Wercker **Workflows** using the console. Click **Add New Pipeline** button.

- Name: deploy
- YML Pipeline name: deploy
- Hook type: Default

2. Click Create. Click again Workflows tab.

3. Click **+** sign on the right side on **build** pipeline. It opens *When pipeline build finishes* dialog, select:

- Execute pipeline: deploy

4. Click Add. Now your build and deploy pipelines are connected in a continuous workflow. Any push to the master code repository will trigger the complete CI/CD workflow.

5. We defined environment variables in Wercker for our database connection: 

- DB_USER
- DB_PASSWORD
- DB_HOST
- DB_SERVICE

6. Change the application code in **promotion.py** to use these variables. Import **os** library:

    ````
    ...
    from bottle import Bottle, run
    import cx_Oracle
    import os
    ...
    ````

7. Change the code in the '\_\_main__' section to use these OS variables:

    ````
    ...

    if __name__ == '__main__':
        DBUSER = os.getenv('DB_USER')
        DBPASS = os.getenv('DB_PASSWORD')
        DBHOST = os.getenv('DB_HOST')
        DBSERV = os.getenv('DB_SERVICE')
    ...
    ````

8. Commit and push these application changes to the master repository.

    ````
    git commit -a -m "Use environment variables for DB connection"
    git push
    ````

9. Verify on Wercker console, under Runs, the complete workflow build + deploy is successful.

10. From the command line, check if the deployment was successful.

    ````
    kubectl get deployments
    NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
    kubernetes-bootcamp   1/1     1            1           11m
    orcl-ws-app           1/1     1            1           3m34s
    ````

## **Step 3:** Run Web Service

1. Open Kubernetes Dashboard, login using the token, and check the deployment on Overview under **Deployments**. Go down on the page, and view our Python microservice under **Services**, **orcl-ws-app**. Copy the IP address of this microservice under External endpoints, [orcl-ws-app-IP]. 

2. Open a browser on your laptop, and navigate to [orcl-ws-app-IP]/conn. The response is '19.6.0.0.0'. Your Python web micro service application is connected to your Oracle Database and it returns the Database version.

3. In your browser open [orcl-ws-app-IP]/salary_increase/8. It simulates a salary increase with 8% for all employees in our HR schema. 

4. Now open [orcl-ws-app-IP]/add_commission/.15. This web service simulates adding 15% to the commission for all employees. 

5. On Kubernetes Dashboard, on Overview, under Pods, see on which node is deployed your application (10.0.0.XX). Navigate to Cluster > Nodes, and click on this node. Get this node **ExternalIP** next to Addresses.

6. Connect to the OKE node:

    ````
    ssh -C -i ~/orcl-ws-cicd/keys/id_rsa opc@[node_ExternalIP]
    ````

7. Show the ID of the latest container deployed on the node.

    ````
    docker ps -l -q
    ````

8. Connect to this last container deployed (this is our Python micro web service platform) using shell command line.

    ````
    docker exec -it [container_ID] sh
    ````

9. List files on this container.

    ````
    ls
    README.md     kubernetes_deployment.yml.template			  promotion	     wercker.yml
    __pycache__   kubernetes_service.yml.template				  promotion.py
    activate_env  oracle-instantclient19.6-basiclite-19.6.0.0.0-1.x86_64.rpm  requirements.pip
    keys	      orclvenv							  test_promotion.py
    ````

10. Close both container and node connections.

    ````
    exit

    exit
    ````

## Acknowledgements

- **Author** - Valentin Leonard Tabacaru
- **Last Updated By/Date** - Valentin Leonard Tabacaru, Principal Product Manager, DB Product Management, May 2020

## Need Help?
Please submit feedback or ask for help using our [LiveLabs Support Forum](https://community.oracle.com/tech/developers/categories/livelabsdiscussions). Please click the **Log In** button and login using your Oracle Account. Click the **Ask A Question** button to the left to start a *New Discussion* or *Ask a Question*.  Please include your workshop name and lab name.  You can also include screenshots and attach files.  Engage directly with the author of the workshop.

If you do not have an Oracle Account, click [here](https://profile.oracle.com/myprofile/account/create-account.jspx) to create one.

