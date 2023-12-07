#  Distributed Fine-Tuning of Language Models (LLMs)

**Expected Demo Time: 15min**

This demo demonstrates how to perform fine-tuning of large language models (LLM) in
a distributed manner.

## When to use this demo?

This demo is perfect when:

* The customer has some hands-on experience with Language Models (LLMs), especially in using them for tasks like training or tweaking or, at least, making predictions.

* The top priority is making these language models work even better for specific jobs or tasks by fine-tuning them.

### When NOT to use this demo?

This demo is not suitable when:

* The customer hasn't begun working with Language Models (LLMs) yet and is merely considering the possibility of using them. In such cases, it's recommended to focus on demonstrating the Hugging Face step.

## Video
[![Watch the video](https://cdn.loom.com/sessions/thumbnails/75fbc94db88d41aa93649df47b4976ea-with-play.gif)](https://www.loom.com/share/75fbc94db88d41aa93649df47b4976ea)

## How to demo?

> We have an existing project that's connected to the right Git repository and allows you to demo this easily.

### Before the demo:
Create a new Valohai project and connect it the [valohai-llms](https://github.com/SofiaChar/valohai-distributed-llms) repository.
   * You can also use [this project](https://app.valohai.com/p/SharkOrg/llms-distributed-example/)

#### Overview:
We are fine-tuning **Bert Large CNN** model with summarization dataset from Samsung called **SAMSum**.

The repo consists of three main scripts that enable distributed training:
* Distributed training within one machine (should have more than 1 GPU):
  * **Torchrun - the feature from Pytorch**. 
    * Doesn't really need any changes to the regular training script.
    * In our case we use transformers.Trainer - distribution is managed fully by torchrun.
  * **Accelerate - the tool from Hugging Face - Accelerator**.
    * Requires some changes to the regular training script (Acelerator specific - **NOT Valohai specific**)
    * We use the regular training loop.
* **Distributed training between few machines (one GPU per each is fine)**:
  * Uses valohai.distributed feature. Some extra code - Valohai specific - is required.
  * We have to do data partition (divide the dataset into parts, subset per machine)
  * We calculate average gradients (between machines) after each step of the training.

### Demo :popcorn:

#### Beginning - Showing completed execution - Fine-tuning LLM within one machine:
We are fine-tuning **Bert Large CNN** model with summarization dataset from Samsung called **SAMSum**.
Although Valohai does not care which model of framework you are using.

1. Show the pinned execution `train-torchrun`. This job does distributed fine-tuning of LLM **within one machine with 4 GPUs**. 
2. General info about the scrpit:
   * We load the dataset from S3 bucket
   * The model comes from Hugging Face in this case; It can also be loaded from external data sources like S3 bucket.
   * We save the logs to Valohai, so it can be visualized in Metadata tab.
   * The fine-tuned model is saved in the Outputs tab.
     * Optional: The model is also saved as Valohai Dataset (llm-models) with an alias.
3. Go through the Details tab:
   * We use powerful and expensive machine with lots of memory, with 32 CPUs and 4 GPUs:
     * Mention that this type of machines are often hard to get - high demand. Sometimes you can wait few hours to get the machine.
   * The distribution is done with torchrun by Pytorch - **Nothing Valohai specific** is needed in the code.
   * Show that training the model for one epoch took 25 minutes (which is fast for LLM) and the price is ~5.5$
Mention that this approach can be used to distribute between GPUs on your on-prem machine.

####  Fine-tuning LLM between few machines:
1. Create Task using the `train-task`. 
    * Task type - Distributed. 
    * Execution count set to 4.
      * Mention that with this parameter it's easy to make experiments, distribute among as many machines as needed.
2. Show that now we use much cheaper machine with one GPU.
   * Mention that depending on the GPU type and amount of memory, the training can take longer, but these machines are available 90% of the time.
3. Mention that in the script we are using `valohai.distributed` and `torch.multiprocessing`. You need to add few lines of code to set up the IPs, the port to enable distributed training.
4. Show the execution when the training have started 
   * Show this `3/4 members in distributed group after 2.23 seconds... All 4/4 members announced themselves ...`
     * The members of the distributed group are connected
   * Show the line in the logs where we map the data - we have 14732 samples.
   * Then show the training - on this machine we have use 3683 training samples - data partition has been done.
5. Show the estimate time for the fine-tuning (~3 hours), in the case of distributing between machines it takes longer, than within one machine.

## FAQ?
#### Q1: Why distributed is important/relevant for fine-tuning LLMs?

Using distributed computing is important for fine-tuning Large Language Models because it makes the process faster, more efficient, and capable of handling the large amounts of data and complex computations involved in training these models.

#### Q2: Which Distributed Frameworks are Supported?

Answer: Valohai supports various distributed training frameworks, providing flexibility to users based on their preferences and requirements.
The tools that we have tested with LLMs are Torchrun and Accelerate. You can also use Tensorflow, Horovod etc.
[Examples to share](https://github.com/valohai/distributed-examples) for distributed training on different frameworks.

#### Q3: Is Data Distributed Across Workers?

Yes, users have the option to distribute data across workers in Valohai. In the distributing between machines case, every worker downloads the complete dataset from Valohai inputs. Subsequently, each worker engages in training on a specific segment of the partitioned dataset. The scripts, `train-torchrun.py` and `train-accelerator.py`, manage the data partition through distribution tools.
#### Q4: Why Distributed Computing for LLMs:

When is it essential to use distributed computing for fine-tuning LLMs?

Distributed computing is essential when dealing with the computational intensity, resource efficiency, and scalability required for training large language models efficiently.

Distributed computing is essential for fine-tuning LLMs when:
* Handling large language models with a massive number of parameters.
* Seeking computational efficiency and accelerated training.
* Processing large datasets efficiently through parallel processing.
* Addressing memory requirements exceeding the capacity of a single GPU or machine.
* Scaling training for larger language models.

#### Q5: Cost and Duration:
In a comparison of using different machines for fine-tuning with the same data and parameters:

**p3.8xlarge (GPU: Tesla V100) - distribute within one machine:**
* Price: $13/h
* Time Taken: ~25min
*  Bill: $5.5

**p3.2xlarge (GPU: Tesla V100) - distribute between few machines:**
* Estimated Time: ~2.5h
* Price: ~$3/h
* Bill: $7.5/machine = $30

**g4.xlarge (GPU: Tesla T4) - distribute between few machines:**
* Price: $0.5/h
* Estimated Time: ~4h
* Estimated Bill: ~$2/machine = $8

For the case when we distribute between few machines like `p3.2xlarge` or `g4.xlarge`, we have longer training time and spend more money.
Let's see the factors affecting performance:

* Communication Overhead: Distributing between machines introduces communication overhead as data needs to be transferred between them. This can lead to delays in synchronization and hinder overall training speed.
* Data Transfer Latency: The need to transfer data between machines results in latency, impacting the training process. Small batch sizes and frequent data exchanges contribute to slower convergence.
* Resource Isolation: Each machine operates independently with its own GPU. This isolation can lead to suboptimal resource utilization and slower convergence compared to a unified, multi-GPU setup.

For the `p3.8xlarge` (Distributing within one machine between few GPUs):

Factors Affecting Performance:

* Unified Memory Access: In a single-machine, multi-GPU setup, all GPUs have unified access to the system memory. This allows for efficient data sharing and reduces the communication overhead compared to distributing between machines.
* Parallel Processing: GPUs within the same machine can perform parallel processing more seamlessly as they share a common memory space. This results in faster model updates and shorter training times.
* Coordinated Synchronization: Synchronization among GPUs in a single machine is more coordinated and efficient compared to communication between separate machines. This leads to faster convergence and reduced training times.


#### Q6:  Master IP and Valohai Library:

- Explain the significance of obtaining the master IP.
- Do you always need to use the Valohai library for obtaining the master IP?
  - **Answer:** The master IP is crucial for coordinating distributed training. While the Valohai library can be used (`valohai.distributed.master()`), the master IP can also be obtained from our JSON file during the execution.

#### Q7: Distributed Computing Concepts (Rank and Averaging Gradients):

- **Rank:** Refers to the unique identifier assigned to each process or machine in distributed computing - in our case it's the machine.
- **Averaging Gradients:** In distributed training, it involves aggregating gradient updates from different machines to update the model parameters. So, on each step of the training we aggregate the gradients from all machines.

#### Q8: UI Execution Count and Code Changes:

Are code changes necessary when adjusting the execution count?
  - Changing the execution count in the Valohai UI does not necessarily require code changes. For this demo you don't need to change any code to update the number of machines/GPUs to distribute between.
    - For `train-torchrun.py` and `train-accelerator.py` you only need to change the parameter `--nproc-per-node=4` and `--num_processes=4` respectively in command section when creating the execution.
    - For `train-task.py` change the `execution-count` when creating a Task.

#### Q9: Huggingface Model Usage:

Explain the importance of the "bart-large-cnn" model from Huggingface within the code.
- The tokenizer and pretrained model is loaded from Huggingface by using these lines of code:
```
tokenizer = AutoTokenizer.from_pretrained(model_ckpt)
pretrained_model = AutoModelForSeq2SeqLM.from_pretrained(model_ckpt)
```
- You can path another model from Huggingface as a parameter.