# Introduction
This repo demonstrates how to run and benchmark inferencing for llama3.2 models, with 8 GPUs.

## Pre-req
- This assumes that you are using the latest ML-In-a-Box running on A100x8 or H100x8. If you're not running on ML-In-a-Box, please refer to the setup section of https://github.com/gradient-ai/trition-vllm/tree/main for more details.

## Setup tasks

- Spin up the docker for triton-vllm:
```
docker run -ti \
  -d \
  --privileged \
  --gpus all \
  --network=host \
  --shm-size=10.24g --ulimit memlock=-1 \
  -v ${HOME}/newmodels:/root/models \
  -v ${HOME}/.cache/huggingface:/root/.cache/huggingface \
  nvcr.io/nvidia/tritonserver:24.08-vllm-python-py3
```
- Open a separate session tab, and ssh into the machine.
- Run `docker exec -it $(docker ps -q) sh` to shell into the container.
- Run the following commands to install vllm libraries and login to huggingface accounts:
```
pip install git+https://github.com/triton-inference-server/triton_cli.git@0.0.11
huggingface-cli login --token <token> # replace <token> with your HuggingFace Token obtained from https://huggingface.co/settings/tokens
pip install vllm-flash-attn==2.6.2
pip install vllm==0.6.2
```


## Choose the model to run
- Download the zip files in this repo (e.g. llama3.2-1b.zip) and unzip it under `/root/models` directory within the docker container.
- Remove the zip file after unzipping.
- Run `triton start`. For first time running, it will take a few minutes depending on the model size.
- If the triton server starts successfully, You should see something similar to the following lines at the end:
```
#I0603 11:11:10.717657 779 grpc_server.cc:2463] "Started GRPCInferenceService at 0.0.0.0:8001"
#I0603 11:11:10.717897 779 http_server.cc:4692] "Started HTTPService at 0.0.0.0:8000"
#I0603 11:11:10.782444 779 http_server.cc:362] "Started Metrics Service at 0.0.0.0:8002"
```

## Benchmark the model
- Download input.json file and store it under /opt/tritonserver/input.json. Use `wget` or `scp` the file into the machine. `Do not Copy and Paste` the contents of the file as the unicode can get messed up.
- Using perf analyzer to benchmark llama3.2-1b model based on the input.json file 
```
perf_analyzer -m llama3.2-1b --async --input-data /opt/tritonserver/input.json -i grpc --streaming --concurrency-range 512 --service-kind triton -u localhost:8001 --measurement-interval 15000 --stability-percentage 999 --profile-export-file /opt/tritonserver/artifacts/profile_export.json
```
- After every run, please remove the output file by running `rm /opt/tritonserver/artifacts/profile_export.json`
