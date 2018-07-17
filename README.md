# Sort-Reduce

Software library for the sort-reduce algorithm.

## Introduction to sort-reduce

Sort-reduce is an algorithm for sequentializing fine-grained random read-modify-writes, or updates, into secondary storage.

Because most secondary storage devices have coarse, multi-KB, page level access granularities, updating fine-grained values in secondary storage incur a large write amplification.
For example, reading and writing an 8 KB page in order to read and update a 4 byte value results in a 2048-fold write amplification, resulting in a 2048-fold effective performance degradation.

Sort-reduce is an algorithm that mitigates this issue in two parts:
- Log the update requests and sorting it, turning the requests sequential.
- During sorting, whenever update requests directed to the same location are discovered, apply the update between the two requests.

Random updates that benefit from sort-reduce can be formulated like the following:
Updating an array **X** in secondary storage using an update function **f** would look like the following:
For each update request into __index__ using __argument__, X[index] = **f**(X[index], argument).
For example, constructing a histogram would repeat X[index] = X[index] + 1.

Sort-reduce logs and sort the <__index__,__argument__> pair for all update requests, turning it sequential.
In order to reduce the overhead of sorting, whenever two update requests with the same __index__ are discovered, **f** is applied between two __argument__s, reducing two update requests into one.

More details about the sort-reduce algorithm, as well as its application to graph analytics, can be seen in the paper [GraFBoost: Accelerated Flash Storage for External Graph Analytics](http://people.csail.mit.edu/wjun/papers/isca2018-camera.pdf).


## Building the library

The library can be either built into a .a file or be used by building and linking its .cpp file.

libsortreduce.a can be built simply by running **make**

## Usage

The library is templatized with the array index (key) and argument (value) types.
For example, for 64-bit keys and 32-bit values, the library can be instantiated like the following:

```
SortReduceTypes::Config<uint64_t,uint32_t>* conf = new SortReduceTypes::Config<uint64_t,uint32_t>("/mnt/ssd0/scratch/", "output.dat", 16);
conf->SetUpdateFunction(&update_function);
conf->SetManagedBufferSize(1024*1024*32, 64); // 4 GB

SortReduce<uint64_t,uint32_t>* sr = new SortReduce<uint64_t,uint32_t>(conf);
```

The above code uses "/mnt/ssd0/scratch/" for temporary and output directories, allows up to 16 threads to be used, and stores the sort-reduced array at "/mnt/ssd0/scratch/output.dat".
The user-defined update function is given as update_function, and assigns 4 GB of memory (64x 32 MB blocks).
Block size will be automatically adjusted in the future. For now, 32 MB is a good number.

An example update function for building histograms can be seen below.

```
uint32_t update_function(uint32_t a, uint32_t b) {
	return a+b;
}
```

Update requests can be applied like as shown below.
Update may return false if there is no buffer space, so we must loop until it doesn't.
Once all input is applied, call Finish().

```
for ( uint64_t i = 0; i < element_count; i++ ) { 
	while ( !sr->Update(key, 1) ){}
}
sr->Finish();

```


### Building

See example makefiles in examples/simple

### Memory allocation

### Thread allocation

### Multi-thread ingestion

### More types

## Implementation details