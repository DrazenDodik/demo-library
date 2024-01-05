# Launch a collection of pipelines for different models in Valohai

This demo shows how to easily
- Run hyperparameter tuning as a part of a pipeline
- Scale pipelines to train the same model on different datasets

## When to use this demo
The customer wants to:
- Retrain their model for different contexts (multiple patients, devices, vehicles..)
    - "We have a model built, that is trained for device xyz123, but now we have to fit the model for 100s of other devices"
- Run parallel steps, e.g. hyperparameter tuning, inside a pipeline
- Train the model for different environments (e.g. edge, online, mobile...) or compare multiple variations of the model in a pipeline

## When not to use this demo
The customer:
- Has heavy focus on exploration using notebooks and doesn't use an IDE or benefit from dividing their work into separate pure Python scripts that could be combined into a pipeline.
- Doesn't need to take their work into production or cannot define who/what will use their model.

## Video

*video content*

## How to demo?
- Start by showing a completed pipeline and talking about the fundamentals.
    - See the cheat sheet below.  
- Go to the YAML file in Github and show the pipeline, mention that you will share it after the call. 
    - DISCLAIMER: If the audience consists mainly of ML / DevOps directors or the likes, it makes sense to skip showing the YAML to avoid making Valohai feel complicated. 
- Back in the UI, show how to create parallel pipelines by using the pipeline level parameters.
    - Change the value of the pipeline parameter called "dataset". Available values:
        - all_harbors
        - harbor_A
        - harbor_B
        - harbor_C
- Open the preprocess node and show how to change inputs.
- Open the train node and show how to change the parameter values.
- Show how to create a trigger to run the pipeline for production runs (hourly, daily, monthly, etc.)

### Fundamentals cheat sheet
- Each node is "individual and isolated", i.e. they run on individual machines and in isolated Docker containers. 
- Each node can run on the same instance type or you can select different type for each node if needed (e.g. CPU vs GPU workloads), Valohai will handle scaling the right machine.
- Each node is versioned: information about the environment (machine and Docker image), inputs, parameters, who ran the job...
- Valohai versions the outputs of each node, and handles passing them as inputs to the next nodes 
    - Valohai handles authentication, authorization, downloading and caching of input files. You will only need to tell it which files you want available in your job.
    - From your codes point of view all these files are local.
- We’re using datasets that have a collection of images. 
    - You can show images from Details tab: Click on the eye symbol for the input to show the preview.
        - Note that this requires using `s3://valohai-demo-library-data/dynamic-pipelines/train/images/*` as the input for preprocessing node, default is a .zip file. 
- Valohai has a dataset function, a versioned collection of files with human readable names, easy to share across team and update (code, or UI). 
    - Even if we use “latest” Valohai will version the exact files that were pulled with the data store URLs, it won’t just say “latest” under the Details page.
    - Dataset is used as an input in the train node.
- Parameters are either hyperparamerts, or any configuration value (e.g. ship id, patient id)
    - We can easily then run multiple pipelines with a collection of these different parameters (100 ships ⇒ 100 pipelines) or dynamically scale up/down our pipeline to say (1 pipeline but the number of jobs inside it will depend on number of ships). Just depends do you want to isolate each job or fan out/in all jobs.
- Metadata can be used for graphing purposes or for edge conditioning (stop pipeline if metadata exceeds certain value).
    - See train node, plot for example epoch vs accuracy and/or loss.


## Acknowledgements
The example here is based on the Ship classification Notebook by [Arpit Jain](https://www.kaggle.com/code/arpitjain007/ship-classification/notebook). 
- The dataset is available [here](https://www.kaggle.com/datasets/arpitjain007/game-of-deep-learning-ship-datasets).