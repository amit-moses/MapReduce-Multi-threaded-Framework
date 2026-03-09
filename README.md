# MapReduce Multi-threaded Framework

## Project Overview

This project implements a **robust multi-threaded MapReduce framework in C++**.  
It provides a **generic infrastructure for parallel data-processing tasks** using the MapReduce paradigm.

The framework abstracts the complexities of **thread management, synchronization, workload distribution, and execution phases**, allowing developers to focus only on implementing the **Map and Reduce logic** for their tasks.

By utilizing **multi-core processing**, the framework significantly reduces runtime for large-scale computations compared to single-threaded execution.

---

## Key Features

### Generic MapReduce Architecture

Supports any task defined by custom **Map** and **Reduce** functions implemented by the user.

### Asynchronous Execution

Jobs run **asynchronously in the background**, returning a `JobHandle` that allows non-blocking monitoring of the job's progress.

### Advanced Synchronization

Ensures thread safety using:

- **Atomic counters**
- **Mutexes**
- **Barriers**

### Dynamic Progress Tracking

Provides **real-time updates** about the job's current stage and percentage of completion.

### Resource Efficiency

Optimized to **minimize memory allocations and data copying**, particularly during the **Shuffle phase**.

---

## Technical Implementation Details

The framework executes the MapReduce workflow in **four distinct stages**.

### Map Phase

Worker threads process input elements in parallel.  
An **atomic counter** distributes work among threads and prevents redundant processing.

### Sort Phase

Each thread **independently sorts its intermediate key-value pairs**, preparing them for the shuffle stage.

### Shuffle Phase

A **dedicated thread** collects and merges the sorted intermediate vectors from all worker threads into **groups of identical keys**.

### Reduce Phase

Worker threads process the grouped keys and generate the **final output results**.

---

## Synchronization Mechanisms

### Atomic State Variable

A single **64-bit atomic variable** manages the job state, including:

| Component       | Purpose                               |
| --------------- | ------------------------------------- |
| Job stage       | Indicates current execution phase     |
| Processed items | Tracks progress                       |
| Total items     | Used to compute completion percentage |

This design enables **fast and thread-safe state updates** without excessive locking.

### Barriers

Barriers ensure that **all threads complete the Map and Sort phases** before the Shuffle phase begins.

### Mutex Protection

A **mutex** protects shared resources such as the **output vector** when threads emit final results.

---

## Execution Pipeline

| Stage   | Description                                                      |
| ------- | ---------------------------------------------------------------- |
| Map     | Threads process input data and emit intermediate key-value pairs |
| Sort    | Each thread sorts its intermediate pairs                         |
| Shuffle | A dedicated thread groups identical keys together                |
| Reduce  | Threads process grouped keys to produce final output             |

---

## API Reference

| Function            | Description                                                                     |
| ------------------- | ------------------------------------------------------------------------------- |
| `startMapReduceJob` | Initializes and starts an asynchronous MapReduce job.                           |
| `waitForJob`        | Blocks until the specified job has completed execution.                         |
| `getJobState`       | Retrieves the current stage and progress percentage of a job.                   |
| `closeJobHandle`    | Safely releases all resources associated with a completed job.                  |
| `emit2`             | Called by the client during the Map phase to emit intermediate key-value pairs. |
| `emit3`             | Called by the framework during the Reduce phase to emit final output pairs.     |

---

## Example Usage

```cpp
class MyClient : public MapReduceClient {
public:

    void map(const K1* key, const V1* value, void* context) const override
    {
        // produce intermediate pairs
        emit2(intermediateKey, intermediateValue, context);
    }

    void reduce(const IntermediateVec* pairs, void* context) const override
    {
        // process grouped pairs
        emit3(outputKey, outputValue, context);
    }
};

int main()
{
    MyClient client;

    JobHandle job = startMapReduceJob(
        client,
        inputVec,
        outputVec,
        multiThreadLevel
    );

    waitForJob(job);
    closeJobHandle(job);

    return 0;
}
```
