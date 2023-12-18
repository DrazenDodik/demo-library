## Distributed Training Modifications

### Train Torchrun
No code changes are required if we are using transformer.Trainer to train our model.

### Train Accelerate

To enable distributed training using the `accelerate` library, several changes have been made to the existing code. The key modifications include the use of `accelerator` to distribute the model and data across multiple GPUs. Here's a description of the changes:

1. **Import Accelerator Module:**
   The `accelerate` module is imported to facilitate distributed training. This includes distributing the model and data across multiple GPUs.

   ```python
   from accelerate import Accelerator
   ```

2. **Initialization of Accelerator:**
   An instance of the `Accelerator` class is created to handle distributed training. This includes setting the device, initializing the model, and preparing the optimizer and data loaders for distributed training.

   ```python
   self.accelerator = Accelerator()
   ```

3. **Model and Optimizer Preparation:**
   The model and optimizer are prepared for distributed training using the `prepare` method of the `Accelerator` class. This involves distributing the model and optimizer across multiple GPUs.

   ```python
   model, optimizer, train_dataloader, eval_dataloader = self.accelerator.prepare(
       self.pretrained_model, optimizer, train_dataloader, eval_dataloader
   )
   ```

4. **Data Parallelism:**
   The training loop is modified to use `accelerator.backward` instead of `loss.backward` to handle distributed training. This ensures that gradients are properly synchronized across GPUs.

   ```python
   self.accelerator.backward(loss)
   ```

5. **Gathering and Synchronization:**
   At various points in the code, the `accelerator.gather` method is used to collect and synchronize metrics or results across all processes. 
Also, the `accelerator.wait_for_everyone` was used to make sure that we gather the metrics from all processes.
  
   ```python
   metrics_list = self.accelerator.gather(torch_metrics)
   self.accelerator.wait_for_everyone()
   ```

6. **Unwrapping the Model:**
   When saving the model or accessing its parameters, the model is unwrapped using `accelerator.unwrap_model` to obtain the original model.

   ```python
   unwrapped_model = self.accelerator.unwrap_model(model)
   ```

These modifications enable the script to run distributed training using the `accelerate` library, leveraging multiple GPUs for improved training speed.


### Train Task - between machines

The provided script has been updated to enable distributed training across multiple machines. Here are the key changes made to facilitate distributed training:

### 1. Import Distributed Modules

```python
import torch.distributed as dist
import torch.multiprocessing as mp
```

### 2. Initialization of Distributed Environment

A distributed environment is initialized using the `dist.init_process_group` method. This includes specifying the communication backend (`nccl` in this case) and setting the rank and world size for each process.

```python
dist.init_process_group(init_method=master_url, rank=my_rank, world_size=world_size, backend='nccl')
```

### 3. Multiprocessing Setup

The script utilizes the `torch.multiprocessing` module to spawn multiple processes. The `init` function is called to set up each process, and the `run` function is executed in each process.

```python
mp.set_start_method('spawn')
p = mp.Process(target=init, args=(url, rank, size, run, my_args))
p.start()
p.join()
```

### 4. Data Partitioning

The `DataPartitioner` class is introduced to partition the dataset among different processes. This class is used to create partitions of the dataset, ensuring that each process gets a portion of the data for training.

### 5. Model and Optimizer Preparation

The model is moved to the specified device (`cuda:0` in this case) and the optimizer is prepared for distributed training.

```python
model = self.pretrained_model.to(self.device)
```

### 6. Training Loop Modification for Distributed Training

The training loop is modified to use the `average_gradients` function, which performs gradient averaging across all processes. This is crucial for synchronizing model updates in a distributed setting.

```python
def average_gradients(model):
    size = float(dist.get_world_size())
    for param in model.parameters():
        dist.all_reduce(param.grad.data, op=dist.ReduceOp.SUM)
        param.grad.data /= size

average_gradients(model)
```

### 7. Synchronization and Aggregation of Metrics

The `synchronize_and_aggregate_metrics` function is introduced to gather and synchronize metrics across all processes. This is necessary for aggregating and calculating the mean of metrics.

These modifications enable the script to run distributed training across multiple machines, leveraging distributed data and model parallelism for improved training speed and scalability.

