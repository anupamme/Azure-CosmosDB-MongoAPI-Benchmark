# YCSB Benchmark for Cosmos DB Mongo API 

This is a **step-by-step** guide to run [YCSB Benchmark](https://github.com/brianfrankcooper/YCSB),  against [Azure Cosmos DB](https://docs.microsoft.com/en-us/azure/cosmos-db/introduction)  [MongoDB API](https://docs.microsoft.com/en-us/azure/cosmos-db/mongodb/mongodb-introduction) . 

## Step 1: Setting up the Azure Cosmos DB Mongo API Environment.

### 1. Create Cosmos DB Mongo API instance

#### 1. Goto [Azure portal](https://ms.portal.azure.com/#create/Microsoft.DocumentDB) and select Azure Cosmos DB API for MongoDB. 

#### 2. Fill in your project details. 

 - In the account name use the name you prefer, we use cosmosdb-mongo-api-benchmarking
 - Select the region where you can also select 4 client VMs (D8 in our case). 
 - Keep all other the values to default 

### 2. Add a new database/collection 
 #### 1. Goto the Cosmos DB instance created in previous step.
 #### 2. Goto **Data Explorer** on the left pane.
#### 3. Select **New Collection** and enter following details
 - Database Name (Create New): benchmarking_dataset
 - Database Throughput: Manual
 - Database Throughput value: 250,000.
 - Collection Id: **usertable** (DO NOT CHANGE THIS NAME)
 - Sharded
 - Shard Key: _id


## 2. Data Loading

### 1. Select a client vm
- We use a 8 core VM because we can run more parallel threads (128 in number).
- In [Azure](https://ms.portal.azure.com/#create/Microsoft.VirtualMachine-ARM), just make sure this VM is in the same region as CosmosDB instance created in the previous step.
- Also use Ubuntu Server 20.04 (you can try others as well)

### 2. Prepare the client vm
- ssh into the client vm
- sudo apt update (to update the package manager)
- sudo apt install openjdk-17-jre
- sudo apt install python

### 3. Prepare YCSB
- curl -O --location https://github.com/brianfrankcooper/YCSB/releases/download/0.17.0/ycsb-0.17.0.tar.gz
- tar xfvz ycsb-0.17.0.tar.gz
- cd ycsb-0.17.0

### 4. Edit the workload file
- open the workload file (vi workloads/workloada) and make these edits:
- recordcount = 10000000 # (10 million records)
- operationcount = 2000000 # (2 million operations)
- readproportion = 1
- updateproportion = 0
- mongodb.url=$connection_string_1_2

### 4. Load the data
- Run YCSB routine to load the data (Takes about an hour):
	- ./bin/ycsb load mongodb -threads 128 -s -P workloads/workloada 
- We chose 128 threads for a 8-core VM, for a 4-core, we found 64 threads were optimal.

## 3. Running the benchmark
### 1. Single VM
 - On the same VM in step 2 once the loading has finished run the following command:
	 - ./bin/ycsb run mongodb -threads 128 -s -P workloads/workloada
- Observe the Operations Per Second (OPS) achieved in this operation. It is expected to be around 5000 OPS.

### 2. Increase the RU allocated in step 1.2
- Use Mongo shell to update the RU allocated to the collection usertable, created in step 1.2. 
- Increase the RU to either 500,000 or 1,000,000. We observed OPS increase linearly with increase in RU allocated. 

### 3. Repeat step 1
- When RU increased to 500,000 we observe 10,000 OPS and when RU set to 1,000,000, we observe 20,000 OPS.

### 4. Spawn multiple VMs
- Spawn 3 more VMs identical to step 2.1.
- Setup these VMs indentical to step 2.1 to 2.4.

### 5. Run concurrent benchmarks
- On all the 4 VMs run the benchmark concurrently as mentioned in step 3.1.
- We observed OPS to be around 20,000 on each machine (when the RU for the collection was set to 1,000,000).