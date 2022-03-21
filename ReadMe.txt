Goal: To run YCSB benchmark against Azure Cosmos DB Mongo API.

Step 1: Setting up the Azure Cosmos DB Mongo API Environment.
    1. Create Cosmos DB Mongo API instance
    2. Add a database e.g. Cosmos_DB_Mongo_API_Benchmarking
    3. Add a collection within the database: usertable (keep the name same) with these characterstics:
        1. partition key: _id
        2. RU allocation: 250,000 dedicated RU (Manual)

Step 2: Spawning a client VM to install YCSB and do data load.
    1. Select a D8 (8 vcpu and 32GB Memory) machine (in the same region as Cosmos DB instance in step 1). This is because we want to spawn multiple threads to max out the RU utilisation.
    2. Download YCSB from their Github repository.
    3. Install prerequired packages: openjdk and python.
    4. Edit the work load file:
        1. Select a workload file e.g. workloads/workloada
        2. Set the value of:
    5. Run the following command to load the data (it will take about an hour):

Step 3: Run YCSB benchmark against the environment on the dataset created in previous step.
    1. When step 2 has finished, run the following command to run the benchmark
    2. Observe the Operations Per Second (OPS) achieved in this operation. In my case it was 5000.

Step 4: Increase the RU allocation for the collection usertable from 250,000 to 1,000,000.
    1. Use the Mongo shell to change the RU allocation.

Step 5: Repeat Step 3.
    2. Observe the Operations Per Second (OPS) achieved in this operation. In my case it was now 20,000.

Step 6: Spawn 3 instances of D8 machines (in the same region) to repeat step 3 concurrently:
    1. Spawn 3 instances of D8 VMs (in the same region) and install YCSB on them (repeat Step 2)
    2. Edit workload files in each of them (same as step 2)

Step 7: Run YCSB parallely from different machines:
    1. Run the following YCSB command from the four machines in parallel and observe OPS, in my case it was 20,000 on each of them.
    2. So I could get 80,000 OPS using this setup.