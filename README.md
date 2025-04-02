# SEAKR: Self-aware Knowledge Retrieval for Adaptive Retrieval Augmented Generation

## Install environment

You need to install our modified vllm, optimized for internal states computation.

```bash
conda create -n SEAKR python=3.10
conda activate SEAKR
cd vllm_embedding
pip install -e .
pip install beir==1.0.1
```

## Set up the retriever

Download the Wikipedia dump from the [DPR repository](https://github.com/facebookresearch/DPR/blob/main/dpr/data/download_data.py#L32) using the following command:

```bash
mkdir -p data/dpr
wget -O data/dpr/psgs_w100.tsv.gz https://dl.fbaipublicfiles.com/dpr/wikipedia_split/psgs_w100.tsv.gz
pushd data/dpr
gzip -d psgs_w100.tsv.gz
popd
```
Use Elasticsearch to index the Wikipedia dump:
```bash
cd data
wget -O elasticsearch-7.17.9.tar.gz https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.17.9-linux-x86_64.tar.gz  # download Elasticsearch
tar zxvf elasticsearch-7.17.9.tar.gz
rm elasticsearch-7.17.9.tar.gz 
cd elasticsearch-7.17.9
nohup bin/elasticsearch &  # run Elasticsearch in background
cd ../..
python build_index.py --data_path data/dpr/psgs_w100.tsv --index_name wiki  --port YOUR_ELASTIC_SEARCH_SERVICE PORT # build index
```

## PREPARE DATASET

For multihop datasets, We use the datasets provided from IRCoT.

### HotpotQA

```bash
mkdir data/multihop_data/hotpotqa
wget -O data/multihop_data/hotpotqa/hotpotqa-dev.json http://curtis.ml.cmu.edu/datasets/hotpot/hotpot_dev_distractor_v1.json
```

### TwoWikiHop

Download the [2WikiMultihop](https://www.dropbox.com/s/ms2m13252h6xubs/data_ids_april7.zip?e=1) dataset from its repository <https://www.dropbox.com/s/ms2m13252h6xubs/data_ids_april7.zip?e=1>. Unzip it and move the folder to `data/multihop_data/2wikimultihopqa`.

### IIRC

```bash
wget -O data/multihop_data/iirc.tgz https://iirc-dataset.s3.us-west-2.amazonaws.com/iirc_train_dev.tgz
tar -xzvf data/multihop_data/iirc.tgz
```
---
For simple QAs, we use the dataset from DPR
### Natural Questions

```bash
mkdir -p data/singlehop_data/
wget -O data/singlehop_data/biencoder-nq-dev.json.gz https://dl.fbaipublicfiles.com/dpr/data/retriever/biencoder-nq-dev.json.gz
gunzip data/singlehop_data/biencoder-nq-dev.json.gz
```

### TriviaQA

```bash
wget -O data/singlehop_data/biencoder-trivia-dev.json.gz https://dl.fbaipublicfiles.com/dpr/data/retriever/biencoder-trivia-dev.json.gz
gunzip data/singlehop_data/biencoder-trivia-dev.json.gz
```

### SQuAD
```bash
wget -O data/singlehop_data/biencoder-squad1-dev.json.gz https://dl.fbaipublicfiles.com/dpr/data/retriever/biencoder-squad1-dev.json.gz
gunzip data/singlehop_data/biencoder-squad1-dev.json.gz
```

## Run And Evaluate

```bash
conda activate __YOUR_ENVIRONMENT__

CUDA_VISIBLE_DEVICES="DEVICES_NUMBER" python main.py \
    --n_shot 10 \
    --retriever_port YOUR_ELASTIC_SERVICE_PORT \
    --dataset_name hotpotqa \
    --eigen_threshold -6.0 \
    --save_dir __YOUR_SAVE_DIR__ \
    --model_name __YOUR_MODEL_SAVED_DIRECTORY__ \
    --served_model_name __YOUR_MODEL_NAME__ \
    --max_reasoning_steps 7 \
    --max_docs 5
```

It will generate a `results.jsonl` in the saved directory. And you can input the file path to the notebook `eval.ipynb`. # SeaKR-Anon
