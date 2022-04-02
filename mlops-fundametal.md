# Kubeflow

## Describing a Kubeflow pipeline with KF DSL
Kubeflow offers a *Domain Specific Language* in Python that allows us to use Python code to describe Kubeflow tasks in a DAG (like Airflow)

For example
```python
import kfp

@kfp.dsl.pipeline(
    name='...',
    description='...'
)
def our_pipeline_function(
    arg1,
    arg2,
    ...
)
```
A Kubeflow Pipeline function must first be decorated with the `
@kfp.dsl.pipeline` decorator. We will name the pipeline and also describe what it does (via `name` and `description`). This metadata is then available through the UI when we upload the pipeline. Next we choose the arguments the pipeline will takes as input function (`arg1`,`arg2`, etc) and these arguments will become the pipeline run parameters in the Kubeflow UI.

### Define tasks (of DAG) within the pipeline function body

1. In the first step we instantiate the Tasks as **Kubeflow OPs/components** to be composed. 
    - These OPs represent Docker containers that are executed when the tasks are run. 
2. In the second step, we composed these OPs. That is, we specify the order to run these tasks (just like Airflow)

For example,
```python
train_model = mlengine_train_op(
    ...
)

eval_model = evaluate_model_op(
    ...
    model_path = str(train_model.outputs['job_dir']), # pass the output of the first OP to the input of the second OP
    ...
)
```
Here we have 2 OPs components: `mlengine_train_op` and `evaluate_model_op`. And we pass the output of the first OP to the input of the second OP.

More advanced, some OPs can be triggered **conditionally** to other OPs output, in that case, we could use 
```python
with kfp.dsl.Condition(...):
    # do some work
```

## Kubeflow component
A KF component or **pipeline component** is self-contained set of code that performs **one step** in the ML workflow (pipeline). Each component in a pipeline executes **independently**. The components **DO NOT** run in the same process and **CANNOT directly** share in-memory data. And usually, we must package your component as a **Docker image**.

A component is normally composed of:
- The component **code**, which implements the logic needed to perform a step in workflow.
- A component **specification**
    - `metadata`: its name and description
    - `interface`: the component’s **inputs** and **outputs**
    - `implementation`: the Docker container image to run, how to **pass inputs** to our component code, and how to **get** the component’s **outputs**.

There are 3 main types of Kubeflow components:
- Pre-built components
- Lightweight Python components
- Custom components

## Pre-built components
Load and use, for example
```python
import kfp

URI = 'path/to/repo/components'

# Create store
component_store = kfp.components.ComponentStore(local_search_paths=None, url_search_prefixes=[URI])

# Load component from store
mlengine_train_op = component_store.load_component('to/folder/contains/component.yml')

# Initiate to use
train_model = mlengine_train_op(
    ...
)

# OR ...
mlengine_train_op = kfp.components.load_component_from_url(
    'path/to/repo/components/which/contains/component.yml'
)
# then initiate to use
...
```
References:
- [kfp.components._component_store](https://kubeflow-pipelines.readthedocs.io/en/stable/_modules/kfp/components/_component_store.html)
- [kfp.components._components](https://kubeflow-pipelines.readthedocs.io/en/stable/_modules/kfp/components/_components.html)

## Lightweight Python components
We wrap Python functions into Kubeflow components in the case where we **DON'T want** to write Dockerfile, build & push Docker images into some Container Registry (because they're just too small functions for ex). 

KF SDK helps us by offering 
 - `func_to_container_op` function
 - `create_component_from_func` function 

They are equivalent, but `func_to_container_op` **is deprecated** in v2 of the SDK
```python
from some_helper import func_to_wrap_1

from kfp.components import create_component_from_func

func_op_1 = create_component_from_func(
    func=func_to_wrap_1,
    output_component_file='component.yaml', # This is optional. It saves the component spec for future use.
    base_image='tensorflow/tensorflow:1.11.0-py3', # if you do not specify a container image, your Python-function based component uses the python:3.7 container image by default
    packages_to_install=['pandas==1.1.4']
)

# OR ...
from kfp.components import func_to_container_op # deprecated in v2 SDK

func_op_1 = func_to_container_op(
    func_to_wrap_1,
    base_image='python:3.7'
)
```

## Custom components
In the case where the task is become more complex for example, we may need to use custom components. Technically, we have to follow several steps to achieve that

1. Write our own code in any language we want because we're in charge of writing our own container
2. Write a Dockerfile to package this code
3. Build and push the Docker image to some container registry
4. Write a yaml description file (`component.yml`) that essentially specify two things
    - the URL of the corresponding image on container registry
    - the run parameters to the component
5. Use this description file to load component into the pipeline

## Compile and Run pipeline

### Option 1: Compile and then upload in UI
```python
# 1. To compile pipeline and save it as pipeline.yaml
kfp.compiler.Compiler().compile(
    pipeline_func=my_pipeline,
    package_path='pipeline.yaml')

# 2. Using the Kubeflow Pipelines User Interface to upload and run pipeline.yaml  
``` 

### Option 2: Compile and deploy using CLI
```shell
$ dsl-compile --py <path/to/pipeline>.py --output <output_filename_for_future_use>.yaml
$ kfp --endpoint $ENDPOINT pipeline upload -p <pipeline_name> <output_filename_for_future_use>.yaml
```
:information_source: For `ENDPOINT`, use the value of the `host` variable in the **Connect to this Kubeflow Pipelines instance from a Python client via Kubeflow Pipelines SDK** section of the **SETTINGS** window

:fast_forward: To **submit the run** using KFP CLI
```bash
# Find the pipeline id appropriated via
$ kfp --endpoint $ENDPOINT pipeline list

# Run the pipeline using the kfp command line 
$ kfp --endpoint $ENDPOINT run submit \
-e <EXPERIMENT NAME> \ # choose any name you want
-r <RUN ID> \ # an arbitrary name
-p <PIPELINE ID> \
arg1=...\ # arguments accepted by pipeline
arg2=...\ 
arg3=...\
...
``` 

### Option 3: Run the pipeline using Kubeflow Pipelines SDK client
```python
# 1. Create an instance of the kfp.Client class
client = kfp.Client() # change arguments according
                      # When not specified, `host` defaults to env var KF_PIPELINES_ENDPOINT.

# 2. Run the pipeline
client.create_run_from_pipeline_func(
    my_pipeline,
    arguments={
        ...
    })
```

## Pipelines SDK
Nowadays, Kubeflow offers us two modes of Pipelines SDK
- [Pipelines SDK](https://www.kubeflow.org/docs/components/pipelines/sdk/sdk-overview/) - stable
- [Pipelines SDK v2](https://www.kubeflow.org/docs/components/pipelines/sdk-v2/v2-compatibility/) - beta
    -  Starting with **Kubeflow Pipelines 1.6**, you can build and run pipelines in v2 compatibility mode.
