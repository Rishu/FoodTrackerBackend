# Deploying the Kitura FoodServer to IBM Cloud

### Pre-Requisites:

**Creating the Food Tracker**  
This tutorial follows on from the FoodTracker Application and server created by following the [FoodTrackerBackend](https://github.com/IBM/FoodTrackerBackend) tutorial.

If you have not completed the [FoodTrackerBackend](https://github.com/IBM/FoodTrackerBackend) tutorial go to the [CompletedFoodTracker](https://github.com/IBM/FoodTrackerBackend/tree/CompletedFoodTracker) branch and follow the README instructions.

**Create a free IBM Cloud Account**  
Go to the following URL, fill out the form and press "Create Account":  
https://console.bluemix.net/registration?cm_sp=dw-bluemix-_-swift-_-devcenter

**Install the IBM Developer Tools**  
Run the `idt` command of the Kitura CLI:
```
kitura idt
```

**Obtain a GitHub ID**  
Go to the following URL, enter Username, Email and Password and press "Sign up for GitHub":  
https://github.com/

**Install the Git CLI**  
`brew install git`  

## Use ElephantSQL to host a PostgreSQL database
So far, our FoodTrackerServer has been using a local PostgreSQL database to store the meals. Once the server is hosted in the cloud it will need to query an online database. Fortunately, IBM offers a PostgreSQL database hosting service called ElephantSQL.

1. Create an ElephantSQL service:  
    i. Add an ElephantSQL service from the [IBM Cloud Dashboard](https://console.bluemix.net/catalog/services/elephantsql).  
    ii. Select the Tiny Turtle plan for a free to use database and click "Create".
2. Get the database URL:  
    i. Click "open ElephantSQL dashboard".  
    ii. Inside the dashboard details section copy the URL.  
    URL structure: `postgres://userid:pwd@host:port/db`

3. Change your applications database:  
    i. Open the Sources > Application > Application.swift file
    ii. Replace your `pool` initialiser with:
    ```
    let pool = PostgreSQLConnection.createPool(url: URL(string: "<your URL here>")!, poolOptions: ConnectionPoolOptions(initialCapacity: 1, maxCapacity: 3, timeout: 10000))
    ```
    Replacing &lt;your URL here&gt; with the database URL.
4. Update Dockerfile and Dockerfile-tools for postgreSQL:  
    i. Go to the root directory of your FoodTrackerServer project
    ```
    cd ~/FoodTrackerBackend/FoodServer/
    ```
    ii. Open the Dockerfile and open Dockerfile-tools
    ```
    open Dockerfile
    open Dockerfile-tools
    ```
    iii. In both files, replace the line:
    ```
    # RUN apt-get update && apt-get dist-upgrade -y
    ```
    With:
    ```
    RUN apt-get update && apt-get dist-upgrade -y && apt-get install -y libpq-dev && apt-get clean
    ```
    This tells Docker to install the postgreSQL library to the container. If during Kitura create you selected postgreSQL as a service this would be done for you.


## Building and Testing on Linux
Before deploying to the cloud, it is useful to be able to build and test the Kitura application on Linux to ensure that it will compile and run when deployed. The Kitura CLI uses three features of the IBM Developer Tools: `build`, `run` and `test` to allow you to easily verify your application locally.

### 1: Build your Kitura application locally for Linux
1. Go to the root directory of your FoodTrackerServer project  
   ```
   cd ~/FoodTrackerBackend/FoodServer/
   ```
2. Build your Kitura application for Linux:
   ```
   kitura build  
   ```
   This builds your Kitura application using a Linux Ubuntu 14.04 Docker container locally on your laptop, ensuring that the application will compile successfully when deployed to the cloud.
3. Run your Kitura application on Linux
   ```
   kitura run
   ```
   This runs your Kitura application inside the Linux Docker container, verifying that it will run when deployed to the Cloud. Once the application is running you can connect to its URLs on localhost as before, eg:
      * SwiftMetrics Dashboard: http://localhost:8080/swiftmetrics-dash
      * Health Check endpoint:  http://localhost:8080/health
4. If you have created tests for your application that are runnable using `swift test` you can also execute those using:
   ```
   idt test
   ```

## Deploying to IBM Cloud
There are two main methods for deploying your application to IBM Cloud:  
1. Using the IBM Developer Tools CLI  
2. Using the IBM Cloud DevOps pipelines  



### Option 1: IBM Developer Tools (IDT)
1. Go to the root directory of your FoodTrackerServer project  
   ```
   cd ~/FoodTrackerBackend/FoodServer/
   ```

2. Log in to IBM Cloud  
   ```  
   bluemix api https://api.ng.bluemix.net
   bluemix login
   bluemix target -o <YOUR_ORG> -s <YOUR_DEV_SPACE>
   ```
   where `YOUR_ORG` is the organisation you used when signing up to IBM Cloud and `YOUR_DEV_SPACE` is the space you created.

5. Deploy your application to IBM Cloud
   ```
   idt deploy
   ```
6. Open your browser to the deployed provided URL to see the application running. You can now update the URL used with KituraKit in the FoodTracker application to connect from the iOS app.

### Option 2: IBM Cloud DevOps Pipelines

In order to use the IBM Cloud DevOps pipelines to build, test and deploy your project, you need to host your project in a Git repository that is visible to IBM Cloud. The easiest way to do this is using GitHub.

#### Create a GitHub project
1. Go to your GitHub account  
   http://github.com
2. Go to your profile by clicking on your avatar in the top right hand corner.
3. Select the "Repositories" tab
4. Select the green "New" button
5. Give your repository a name and press "Create repository"  
**Note:** Keep this page for use later


#### Create a Local Git Project
1. Go to the root directory of your FoodTrackerServer project

```
cd ~/FoodTrackerBackend/Server/FoodTrackerServer/
```
2. Initialise a local git project  
`git init`
3. Add all your files to the project  
`git add -A`
4. Check those files in as a "commit"  
`git commit -m "Initial commit"`
6. Push the commit to GitHub  
Use the two lines under "…or push an existing repository from the command line" from the page displayed when you created your GitHub project.
7. Reload the GitHub project page

#### Create an IBM Cloud DevOps Toolchain for your project

1. Click the "Create Toolchain" button under the "Bluemix toolchain" section of the README.md of your GitHub project.
2. If needed, login to IBM Cloud using your credentials
3. Click the "Create" button
4. Click on the "Delivery Pipeline" tile
5. Wait for the "Deploy Stage" to complete
6. Click the link under "Last Execution Result" to check that the Kitura server is running once its status button turns green.
7. Open your browser to the deployed provided URL to see the application running. You can now update the URL used with KituraKit in the FoodTracker application to connect from the iOS app.
