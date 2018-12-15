#CSC409 EXAM

## Week 1: Distributed Systems

http://www.cs.toronto.edu/~arnold/409/18f/lectures/w01/

#### Goals of Distributed Systems
1. **Scalability**: ability of a system, network, or process, to handle a growing amount of work in a capable manner
   1. add more nodes on available systems
   2. add more data centers
   3. **distribute and replicate data**
      1. **Replicate**: every server has the same data
      2. **Partition**: data split up over multiple servers
   4. make client do work
2. **Performance**: defined by amount of useful work accomplished compared to time and resources used.
   1. **Latency**: the delay before a transfer of data begins following an instruction for its transfer.
   2. **Throughput**: rate of processing work measured as Transactions/Second.
   3. **Fault Tolerance**: ability of a system to behave in a well-defined manner once faults occur.

#### Improving Performance
- More hardware
- Algorithm, Datastructure, Programming Language Optimization
- Multithreading
- Monitoring
- Optimize Database
- Consider Time/Space tradeoffs

```
Cassandra: distributed, wide column store, NoSQL database management system
designed to handle large amounts of data across many commodity servers, providing
high availability with no single point of failure.
```

## Week 2: Multithreading
http://www.cs.toronto.edu/~arnold/409/18f/lectures/w02/

Includes: Tools (curl, ab), [A1 Writeup](https://raw.githubusercontent.com/mehtadarshan1/CSC409A1/master/newBuild/README.md?token=ANGtz1NrbMXPk6sgOeHUTcpqKCYykMxLks5cHYC7wA%3D%3D)

[Multithreaded Break Numeric Java](http://www.cs.toronto.edu/~arnold/409/18f/labs/01/threadding/MD5BreakNumericMT.java)

#### Distributed System Fallacies
* The network is reliable. (this includes hardware, nodes)
* Latency is zero.
* Bandwidth is infinite.
* The network is secure.
* Topology doesn't change.
* There is one administrator.
* Transport cost is zero.
* The network is homogeneous.

**Orchestration**: automated arrangement, coordination, and management of complex computer systems, and services.

## Week 3: Docker
http://www.cs.toronto.edu/~arnold/409/18f/lectures/w03/

Includes: Containers, Services, Dockerfile, Stacks...

## Week 4: CAP Theorem

Consistency, Availability and Partition Tolerance.

- Consistency: All nodes see the same data at the same time. What you write you get to read.
- Availability: A guarantee that every request receives a response about whether it was successful or failed. Whether you want to read or write you will get some response back.
- Partition tolerance: The system continues to operate despite arbitrary message loss or failure of part of the system. Irrespective of communication cut down among the nodes, system still works.

A deeper dive: 
- **Consistency -** A guarantee that every node in a distributed cluster returns the same, most recent, successful write. Consistency refers to every client having the same view of the data. There are various types of consistency models. Consistency in CAP (used to prove the theorem) refers to linearizability or sequential consistency, a very strong form of consistency.
- **Availability -** Every non-failing node returns a response for all read and write requests in a reasonable amount of time. The key word here is every. To be available, every node on (either side of a network partition) must be able to respond in a reasonable amount of time.
- **Partition Tolerant -** The system continues to function and upholds its consistency guarantees in spite of network partitions. Network partitions are a fact of life. Distributed systems guaranteeing partition tolerance can gracefully recover from partitions once the partition heals.

#### Three Groups:
- **CP (Consistent and Partition Tolerant) -** system that is consistent and partition tolerant but never available. CP is referring to a category of systems where availability is sacrificed only in the case of a network partition.
- **CA (Consistent and Available) -** CA systems are consistent and available systems in the absence of any network partition. Often a single node's DB servers are categorized as CA systems. Single node DB servers do not need to deal with partition tolerance and are thus considered CA systems. The only hole in this theory is that single node DB systems are not a network of shared data systems and thus do not fall under the preview of CAP.
- **AP (Available and Partition Tolerant) -** These are systems that are available and partition tolerant but cannot guarantee consistency.

#### ACID in Databases
- Atomicity, Consistency, Isolation, Durability
- https://stackoverflow.com/questions/3740280/acid-and-database-transactions

## Week 5: REDIS
http://www.cs.toronto.edu/~arnold/409/18f/lectures/w05/

Tags: Redis, AOF, Healing, Redis Sentinel, Redis Architecture, 

- RDB: Occassionally, or on demand (SAVE command), store current state of memory in a compact binary format on disk.
- AOF: Continually log all commands executed by this server.

#### RDB Pros and Cons
- \+ RDB compact single-file, good for versioning state of system.
- \+ Disaster recovery: Quickly store small .rdb offsite.
- \+ Good performance: Occassionally redis forks child process to store state of memory. Parent does no disk I/O.
- \+ Disaster recovery: Fast restarts than with AOF.
- \- loose all data since last snapshot
- \- fork may be slow on large datasets impacting performance of server

#### AOF Pros and Cons
- \+ better durability, tunable (via fsync). Note cost of fsync.
- \+ AOF log is append only, so potentially lose only last command
- \- AOF larger than RDB, but has a rewrite/compress option.
- \- AOF slower than RDB (depending on fsync policy)
- \- AOF slower on restarts than RDB

On website:
- Persistence algorithm pseudocode
- Redis Healing methods
- Redis Architecture

## Week 6: Parallel Computing
http://www.cs.toronto.edu/~arnold/409/18f/lectures/w06/

Tags: Process Interactions, Models of Interaction, Apache Spark

- Spark provides is a resilient distributed dataset (RDD), which is a collection of elements partitioned across the nodes of the cluster that can be operated on in parallel

Lecture contains: 
- Sample Spark Code at bottom
- Lab as sample code at bottom
- [Spark Cheat Sheet](https://s3.amazonaws.com/assets.datacamp.com/blog_assets/PySpark_Cheat_Sheet_Python.pdf)
 
#### Solution to [Spark Lab](http://www.cs.toronto.edu/~arnold/409/18f/labs/spark/index.shtml): 

- Load up the genomic data
```
val genome = sc.textFile("/virtual/csc409/sequence2")
```
- Explain what the following does:
```
>> val c = genome.map(_=>1)
>> c.reduce(_+_)

Every element in the genome dataset is mapped to 1. 
The new collection is then assigned to val c.
c.reduce(_+_) collapses the collection by adding every element in the dataset.
```
- Re-write it using the longer form lambda expressions: ```(x,y)=>x+y+z```
```
val c = genome.map(x=>1)
c.reduce((x,y)=>(x+y))
```

- Explain the following:
```
import org.apache.spark._
val rdd = genome.map(line => (SparkEnv.get.executorId,line))
rdd.countByKey()
```
- we map the genome dataset such that it produces a collection containing a tuple of: the ID of the slave (key) and an element from genome. Then in `.countByKey()`, we count the number of elements that each slave has.

- Write a small sequence of scala commands which finds the frequency of all triples. Triples are strings like AAA or GGG, not ATA.
```
genome.map(x => x.split(" ")(3)).map(x=>(x,1)).reduceByKey((x,y)=>(x+y))
```
- Write a small sequence of scala commands which will find the most frequently occurring triple.
```
d.max . . .
```
	
- Write a small scala script which uses all clients and finds all divisors for a number n.
```
val n = 4
val f = genome.map(x => x.split(" ")(3)).map(x=>(x,1)).reduceByKey((x,y)=>(x+y))
f.filter((x => x._2 % n == 0))
```

## Week 7: FAAS / Spark
http://www.cs.toronto.edu/~arnold/409/18f/lectures/w07/

Tags: FAAS, Spark, Openfaas, Spark SQL...

More Faas: http://www.cs.toronto.edu/~arnold/409/18f/lectures/w08/

#### Week 9: AWS
http://www.cs.toronto.edu/~arnold/409/18f/lectures/w09/

Tags: AWS, S3, DynamoDB, Elasticache, Lambda, 

#### Week 10: CUDA
http://www.cs.toronto.edu/~arnold/409/18f/lectures/w10/

Tags: Cuda, Model of Computation

CUDA is a parallel computing platform and programming model invented by NVIDIA. It enables dramatic increases in computing performance by harnessing the power of the graphics processing unit (GPU).

- Write cuda command find which returns "found" if argument e is in array x and "not found" otherwise. The command should have kernel.
```
#include <stdio.h>
#include <time.h>
#define N (1<<20)
#define THREADS_PER_BLOCK 512

__global__ void find(int n, int e, int *x, int *y) {
	int index = threadIdx.x + blockIdx.x * blockDim.x;
	if(x[index] == e){
		*y = 1;
	}
}
void random_ints(int* a, int n){
   int i;

	// Generate a list of random elements
	srand(time(NULL));
   for (i = 0; i < n; ++i) {
		a[i] = rand()%1000;
   }
}

int main(void) {
	int size = N*sizeof(int);
	int *x, *y, e; 

 	// Allocate Unified Memory – accessible from CPU or GPU
  	cudaMallocManaged(&x, size);
  	cudaMallocManaged(&y, sizeof(int));
	
	// Generate random elements
	random_ints(x,N);

	// Going to search for element e
	e = 40;

	// Set y as default 0 - meaning element not found.
	*y=0;

	find<<< N/THREADS_PER_BLOCK, THREADS_PER_BLOCK >>>(N,e,x,y);

  	// Wait for GPU to finish before accessing on host
  	cudaDeviceSynchronize();

	// Print result.
	printf("%d\n",*y);

	// Free memory
	cudaFree(x); cudaFree(y);

	return 0;
}
```

- Write cuda command count which returns the number of times e appears in array x. The command should have kernel.
```
#include <stdio.h>
#include <time.h>
#define N (1<<20)
#define THREADS_PER_BLOCK 512

__global__ void
count (int n, int e, int *x, int *y)
{
    int index = threadIdx.x + blockIdx.x * blockDim.x;
    if (x[index] == e)
    {
        y[index] = 1;
    }
}

void
random_ints (int *a, int n)
{
    int i;

    srand (time (NULL));
    for (i = 0; i < n; ++i)
    {
        a[i] = rand () % 100;
    }
}

int
main (void)
{
    int size = N * sizeof (int);
    int *x, *y, e;

    // Allocate Unified Memory b� accessible from CPU or GPU
    cudaMallocManaged (&x, size);
    cudaMallocManaged (&y, size);

    random_ints (x, N);
    e = 50;
    count <<< N / THREADS_PER_BLOCK, THREADS_PER_BLOCK >>> (N, e, x, y);

    // Wait for GPU to finish before accessing on host
    cudaDeviceSynchronize ();
    int total = 0;
    for (int i = 0; i < N; i++)
    {
        total += y[i];
    }
    printf ("%d\n", total);

    // Free memory
    cudaFree (x);
    cudaFree (y);

    return 0;
}
```
- Write cuda command reverse which reverses x into y

```
#include <stdio.h>
#include <time.h>
#define N (1<<9)
#define THREADS_PER_BLOCK 256

__global__ void reverse(int n, int *x, int *y) {
	int index = threadIdx.x + blockIdx.x * blockDim.x;
	y[n-1-index] = *(x + index);
}

void random_ints(int* a, int n) {
   	int i;

	srand(time(NULL));
   	for (i = 0; i < n; ++i){
		a[i] = rand() % 100;
   	}
}

int main(void) {
	int size = N*sizeof(int);
	int *x, *y, denom; 

 	// Allocate Unified Memory – accessible from CPU or GPU
  	cudaMallocManaged(&x, size);
  	cudaMallocManaged(&y, size);
	
	random_ints(x,N);
	
	if (THREADS_PER_BLOCK > N) {
		denom = N;
	} else {
		denom = THREADS_PER_BLOCK;
	}

	reverse<<< N/denom, denom >>>(N,x,y);

  	// Wait for GPU to finish before accessing on host
  	cudaDeviceSynchronize();

	for (int i=0; i<N; i++){
		printf("%d ", y[i]);
	}

	printf("\n");
	for (int i=0; i<N; i++){
		printf("%d ", x[i]);
	}

	// Free memory
	cudaFree(x); cudaFree(y);

	return 0;
}
```
- Write cuda command max which finds the largest element in x
```
#include <stdio.h>
#include <time.h>
#define N (1<<10)
#define THREADS_PER_BLOCK 512

__global__ void max(int n, int *x, int *y) {
	int index = threadIdx.x + blockIdx.x * blockDim.x;
	atomicMax(y, x[index]);
}

void random_ints(int* a, int n) {
   int i;

	srand(time(NULL));
   for (i = 0; i < n; ++i){
	a[i] = rand()%1000;
   }
}

int main(void) {
	int size = N*sizeof(int);
	int *x, *y; 

 	// Allocate Unified Memory – accessible from CPU or GPU
  	cudaMallocManaged(&x, size);
  	cudaMallocManaged(&y, sizeof(int));
	
	random_ints(x,N);
	max<<< N/THREADS_PER_BLOCK, THREADS_PER_BLOCK >>>(N,x,y);

  	// Wait for GPU to finish before accessing on host
  	cudaDeviceSynchronize();
	printf("%d\n",*y);

	// Free memory
	cudaFree(x); cudaFree(y);

	return 0;
}
```
- Write cuda command oddevensort which sorts array x into array y. Use the following simple sorting algorithm: repeated look left and right and swap (or not) those two elements to make sure they are in order. You will have to think about how to do this and not corrupt memory. What do you think of this sorting algorithm? odd-even sort
```
#include<stdio.h> 
#include<time.h> 
#define N(1 << 4)
#define THREADS_PER_BLOCK 512

__global__ void oddevensort(int n, int * x, int * y) {
  int index = threadIdx.x + blockIdx.x * blockDim.x;
  for (int i = 0; i < n / 2; i++) {
    if ((index % 2 == 0) && (index != n - 1)) {
      if (x[index] > * (x + index + 1)) {
        int temp = x[index];
        x[index] = x[index + 1];
        x[index + 1] = temp;
      }
    } else if ((index % 2 == 1) && (index != n - 1)) {
      if (x[index] > x[index + 1]) {
        int temp = x[index];
        x[index] = x[index + 1];
        x[index + 1] = temp;
      }
    }
  }

  if (threadIdx.x == 0) {
    memcpy(y, x, sizeof(int) * n);
  }

}

void random_ints(int * a, int n) {
  int i;

  srand(time(NULL));
  for (i = 0; i < n; ++i) {
    a[i] = n - i - 1;
  }
}

int main(void) {
  int size = N * sizeof(int);

  int * x, * y;

  // Allocate Unified Memory – accessible from CPU or GPU
  cudaMallocManaged( & x, size);
  cudaMallocManaged( & y, sizeof(int));

  random_ints(x, N);
  for (int i = 0; i < N; i++) {
    printf("%d ", x[i]);
  }
  printf("\n");

  oddevensort << < 1, 16 >>> (N, x, y);
  // Wait for GPU to finish before accessing on host
  cudaDeviceSynchronize();

  for (int i = 0; i < N; i++) {
    printf("%d ", y[i]);
  }
  printf("\n");

  // Free memory
  cudaFree(x);
  cudaFree(y);

  return 0;
}
```
