# End-to-End Memory Network for Goal Oriented Dialogue
This directory contains an implementation of an end-to-end key-value memory network for goal oriented dialog in ngraph.
The idea behind this method is to be able to answer wide range of questions based on a large set of textual information, as opposed to a restricted or sparse knowledge base.

<b>End-to-End Key Value Memory Network</b>
![KVMemNet Pic](https://github.com/siyuanzhao/key-value-memory-networks/blob/master/key_value_mem.png)

# Dependencies:
The primary dependency of this model is NGraph. Download and installation instructions are at http://ngraph.nervanasys.com/index.html/installation.html.
In addition to NGraph, the following libraries are used and can be installed with ```pip install -r requirements.txt```
- numpy
- tqdm

# Files
train_kvmemn2n.py: The primary class which processes the request to train or run inference on the key-value memory network model. It in turn will call the data processing algorithm when necessary.
data.py: Will download and process the data necessary to train the key-value memory network model.
wikiwindows.py: A key data preprocessing stage necessary when using the raw Wikipedia text knowledge base. It is called by data.py, or can be run independently.
model.py: Defines the key-value memory network model.
interactive_util.py: Called by train_kvmemn2n.py to facilitate the interactive mode.


# Dataset
The dataset used for training and evaluation is under the umbrella of the Facebook WikiMovies Dataset tasks (https://research.fb.com/downloads/babi/). The dataset is comprised of questions and answers centered around movies. The files also contain a list of all entities in the dataset such as movie names, people's names, as well as other words which were tagged as entities. The last component of the dataset is a series of knowledge base items as follows:

- Knowledge base. This comes from the Open Movie Database and MovieLens. The questions and answers are based on the KB.
- Raw Wikipedia text on the movies. This text is limited to only the article summary (i.e., its introduction)
- IE performed from the Wikipedia text to build a synthetic KB.

You may choose to use the knowledge base `kb` or raw Wikipedia Text `text` using the argument `--mem_mode`.  The data will be preprocessed and stored according to the `--dataset_dir` and type of knowledge base as given by the argument `--mem_mode`. If you are using `--mem_mode text` the python function `wikiwindows.py` will run which can take a number of hours.  

Please download the tar file from http://www.thespermwhale.com/jaseweston/babi/movieqa.tar.gz and expand the folder into your desired data directory or `--data_dir`. If you are not planning to specify a data directory, please use the path `~/nervana/data`. The code `python train_kvmemn2n.py` will then preprocess the dataset, placing necessary data artifacts into folders within the `--data_dir` directory. The data preprocessing will only need to occur once for each type of knowledge base.  However, you can redo the preprocessing with the argument `--reparse`.

The preprocessing algorithm `wikiwindows.py` can also be ran independently with `python wikiwindows.py ~/nervana/data/` though please note this is not necessary as `train_kvmemn2n.py` will run all data preprocessing steps. To run `wikiwindows.py` independently you need to run `python wikiwindows.py --data_dir ~/nervana/data`


# Training
The base command to train is `python train_kvmemn2n.py`. This algorithm inherits the argument parsing from NGraph, and takes the additional arguments:

`--emb_size` Size of the word-embedding used in the model. (default 50)
`--nhops` Number of memory hops in the network. (default 3)
`--lr` Learning rate (default 0.01)
`--subset` WikiMovies dataset to use for training examples. 'full' or 'wiki-entities' (default 'wiki-entities')
`--reparse` When present, the data preprocessing will be redone
`--mem_mode` The memory source for the model. 'kb' or 'text' (default 'kb')
`--use_v_luts` Run the model using separate value lookup tables for each hop
`--model_file` File path and name to save the model to. When it is not present (default), the model will not be saved
`--inference` Will load a saved model from `--model_file` and run inference without further training
`--restore` Loads a saved model from `--model_file` and continues training, saving the updated weights
`--interactive` Kicks off the interactive session, either at the end of training or with loaded weights in `--inference` mode

Some key arguments from NGraph include:
`--epochs` Number of epochs (default 10)
`--batch_size` Batch size (default 128)
`--data_dir` The directory where the data will be stored (default ~/nervana/data)

The run commands for the results below were:
```
python train_kvmemn2n.py --epochs 2000 --batch_size 32 --emb_size 100 --use_v_luts --model_file path_to_model_dir/kb_model
```
```
python train_kvmemn2n.py --mem_mode text --epochs 2000 --batch_size 32 --emb_size 50 --model_file path_to_model_dir/text_model
```

# Results

The model was trained and evaluated for two different memory modes with the following results:

| Memory Method | This  | Published |  
|------|--------|-----------|
| KB    | 99.96%   | 93.9%     |
| Text (Window-level)    | 67.6%   | 66.8%     |

# Saving and Loading a Model

A model can be saved by setting a path in `--model_file`.  To load and then further train the model add the argument `--restore`. To load the model for inference use the argument `--inference`. The model is currently being saved every 50 epochs.

# Interactive Mode

You can enter an interactive mode using the argument `--interactive`. The interactive mode can be called to launch at the end of training, or direcly after `--inference`. To run inference on the KB model from above we would call:

```
python train_kvmemn2n.py --batch_size 32 --emb_size 100 --use_v_luts --model_file path_to_model_dir/kb_model --inference --interactive
```
Note that we set `--emb_size 100` and `--use_v_luts` as the original model used these parameters.

In this mode you are able to ask a question using either a loaded model, or a model that just completed training. The algorithm will determine if there are any entities in your question (i.e., movies, actors, directors, etc) and pull that entity's key memories. If there is no entity in your question, then the system will politely prompt you to enter a different question.

# References:
- Paper: https://arxiv.org/abs/1606.03126
- Torch Lua implementation: https://github.com/facebook/MemNN, the function wikiwindows.py was taken from that repository and modified for integration into this code base. Modifications are listed at the beginning of the function.