# Iris SKLearn Pipeline

This pipeline is used to demonstrate a basic data science pipeline using the [Iris Dataset](https://scikit-learn.org/stable/auto_examples/datasets/plot_iris_dataset.html) using the sklearn library.

## Prerequisites
- Install [KFP Tekton prerequisites](/samples/README.md)
- Install the additional requirements from [requirements.txt](./requirements.txt)

## Instructions

This sample provides two different ways to execute the pipeline. The first option is to compile the pipeline to a yaml Tekton pipeline, the second option is to connect directly to the Kubeflow Pipeline UI using the kfp TektonClient and run the pipeline directly.

### Compiled

The compiled pipeline uses the kfp-tekton `TektonCompiler()` to generate a yaml object.  The `TektonCompiler()` will produce a Tekton PipelineRun yaml object in the same directory called `iris-pipeline-compiled.yaml`.

This pipeline does utilize a PVC in the pipeline and you may need to set a storage class as an environment variable to match one that is available on your cluster.

To compile the pipeline run:

```sh
# Optional: Set the storage class for the pipeline
export DEFAULT_STORAGE_CLASS="my-storage-class"

python iris-pipeline-compiled.py
```

Once the pipeline is compiled, upload the `iris-pipeline-compiled.yaml` file to the Kubeflow Pipeline dashboard with Tekton Backend.  Once the pipeline is uploaded, you can create a new Pipeline Run from the Dashboard.

### Direct Run

The direct run uses the kfp-tekton `TektonClient()` to connect directly to Kubeflow and create a pipeline run.

Like the compiled pipeline, this pipeline does utilize a PVC in the pipeline and you may need to set a storage class as an environment variable to match one that is available on your cluster.  This example also relies on some additional environment variables to set the Kubeflow UI endpoint and the users bearer token to authenticate to the UI.

To execute the pipeline run:

```sh
# Optional: Set the storage class for the pipeline
export DEFAULT_STORAGE_CLASS="my-storage-class"

DS_PIPELINE_NAMESPACE="my-namespace"
export KUBEFLOW_ENDPOINT="https://$(oc get route ds-pipeline-ui -n DS_PIPELINE_NAMESPACE -o jsonpath='{.spec.host}')"

export BEARER_TOKEN=$(oc whoami --show-token)

python iris-pipeline-direct-run.py
```

A pipeline run will automatically kick off in the UI.
