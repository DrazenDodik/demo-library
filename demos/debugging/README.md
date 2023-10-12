# Debug broken pipelines with Valohai

This demo shows how to run a Valohai execution and attach a local debugger to it, allowing you to easily execute your code step-by-step and debug your issues.

## When to use this demo?
The customer has:
* a heavy focus on engineering
* questions about how Valohai jobs can be troubleshoot and debugged
* a remote development environment, like [workspaces on cnvrg.io](https://app.cnvrg.io/docs/core_concepts/workspaces.html)


## Video

*video content*

## How to demo?

> We have an existing project that's connected to the right Git repository and allows you to demo this easily.

### Before the demo :hourglass:
1. Create a new Valohai project and connect it the [valohai-academy-debugging](https://github.com/DrazenDodik/valohai-academy-debugging) repository.
    * Update the `Fetch Reference` to `*` under Repository settings.
    * You can also use this project that has already been setup.
1. Launch the pipeline `broken-pipeline` **from the main branch** in the project and let it fail.
    * The pipeline fails because there's a condition saying, only continue to the inference step if the metric `precision(B)` is over 0.6.
    * `precision(B)` isn't one of the metrics collected. We'll use this example to debug the code and show we'd add that.
1. Clone the repository to your local machine and open it in VS Code
1. Open `train-model.py` and add a breakpoint on the line that starts with:
    ```python
    model = YOLO("yolov8n.pt")
    ```
1. Make a new branch for example `dd-demo-1`:
    ```bash
    git checkout -b dd-demo-1
    ```
1. Generate a new SSH key with:
    ```bash
    ssh-keygen -t rsa -b 4096 -N '' -f .my-debug-key
    ```
1. In terminal setup the environment and login to Valohai
    ```bash
    # Install virtualenv
    pip3 install virtualenv

    # Setup a virtual environment
    ## Mac users:
    python3 -m virtualenv .venv # mac
    .venv/bin/activate # most users
    . .venv/bin/activate.fish # for fish users

    ## Windows users:
    py -m venv .venv
    .venv\scripts\activate

    # Install the requirements
    pip3 install -r requirements.txt

    # login with your demo credentials
    vh login
    # connect your local directory to the project you created
    # (or connect to the existing one)
    vh project link
    ```

That's it for prep work.

### Demo :popcorn:

#### In Valohai
1. Show a pipeline that ran and failed.
1. Show that our training node completed succesfully but for some reason the next node did not start.
1. Show the pipeline logs and explain that the last node didn't start because there's a condition that was not met.
    * Show the condition in `valohai.yaml`
        ```bash
        if: metadata.precision(B) <= 0.6
        ```
1. Explain that we now want to run just the `train` node again, and attach a debugger to it to it, so we can figure out why it didn't generate any precision metric.
1. Click on the `train` node and copy it into a new execution
    1. Turn on `Run with SSH` and paste in your public key (`.my-debug-key.pub`)
        * Explain that you created the key earlier. You pass the public part of the key to Valohai, and use the private counterpart when connection to it, to authenticate your requests.
    1. Check on the parameter `Debug`
        * Our code defines that if this is set, then it'll wait for the user to attach a debugger before it starts running the training script.
1. Wait for the execution to start and take note of the machine IP that's printed in the logs (in blue)

#### In VS Code
1. From the terminal make an SSH connection to the remote machine. 
    ```bash
    # Replace the YOUR-IP at the end of the line with the machine IP you got from Valohai logs when launching the job
    ssh -i .my-debug-key -p 2222 -L5678:127.0.0.1:5678 YOUR-IP
    ```
1. Explain that we're making an SSH connection:
    * `-i` says we use an identity file `.my-debug-key` which is the private key counterpart of the public key we put into Valohai when we launched the job with SSH enabled.
    * `-p` defines the port that we should use, this was the default value when choosing  Run with SSH
    * `-L` brings the remote machines port 5678 to our machine's port 5678. So when we say "connect to 127.0.0.1 (our machine) on port 5678 it will be routed to the remote machine on the same port"
1. Now open the `train-model.py` file in VS Code
    1. Explain that `debugpy` is a great package that allows us to setup a debugger on port 5678 and say: "If the Debug parameter is set then wait till a debugger is attached before running the code". Otherwise the code might run before we've attached our debugger.
1. Click on the **Run and Debug** on the left-side panel
1. **Run and Debug** with Remote attach, accept the default options (localhost and port 5678)
    * This works because our SSH connection is active and routing localhost:5678 to the remote instance.
1. Your code will execute until it arrives to the line that's loading the pretrained model.
    1. Show in the UI that the job started running.
    1. Back in VS Code, add a new breakpoint to the line that contains
    ```python
        metadata = {
    ```
1. When the next breakpoint is hit hover over `trainer.metrics` to show the available metrics we could have.
1. Show the Variables window on the right to show the different variable values.
1. Use the VS Code `Debug Console` to test that you can access the metric and get some reasonable value.
```python
trainer.metrics["metrics/precision(B)"]
```
1. Deattach the debugger
1. Make changes to your code to add the `precision(B)` metric, like so:
```python
    metadata = {
        "epoch": trainer.epoch,
        "mAP50-95(B)": trainer.metrics["metrics/mAP50-95(B)"],
        "mAP50(B)": trainer.metrics["metrics/mAP50(B)"],
        "recall": trainer.metrics["metrics/recall)"]
        "precision(B)": trainer.metrics["metrics/precision(B)"]
    }
```
1. Save the file and push your changes to your branch
```bash
git add train-model.py
git commit -m "Added precision(B) to Valohai metadata"
git push
```
1. Go the user interface and hit "Fetch repository"
1. Create a new pipeline with this new commit
1. Reuse the `determine` node from the previous pipeline and explain how we don't want to rerun the successful parts. Especially in large pipeline.
1. Run the pipeline
1. Watch it complete successfully :tada:


## Frequently Asked Questions


