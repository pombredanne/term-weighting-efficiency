DeepCT Efficiency Study
=======================

Introduction
------------
This repo shows how to reproduce the experiments from the following SIGIR
short paper: **Efficiency Implications of Term Weighting for Passage Retrieval**
by Joel Mackenzie, Zhuyun Dai, Luke Gallagher, and Jamie Callan.

```
@inproceedings{mdgc20-sigir,
 author = {J. Mackenzie and Z. Dai and L. Gallagher and J. Callan},
 title = {Efficiency Implications of Term Weighting for Passage Retrieval},
 booktitle = {Proc. SIGIR},
 year = {2020},
 pages = {1821--1824}
}

```
 

Tools
-----
We will use the [PISA](https://github.com/pisa-engine/pisa/) and 
[Anserini](https://github.com/castorini/anserini) search systems to build 
indexes of the MS-Marco DeepCT indexes. Please note that the original DeepCT 
codebase can be found [here](https://github.com/AdeDZY/DeepCT).
We also use the [CIFF](https://github.com/osirrc/ciff) tool for index exchange 
between Anserini and PISA. 

These tools are included automatically as git submodules. You may need to run
`git submodule update --init --recursive` to grab all of the tools and their
dependencies.

Next, we'll build these tools. Refer to the README files in the respective
repositories for detailed instructions.

```
echo "Building PISA"
cd pisa
mkdir build
cd build
cmake ..
make
cd ../../
```

```
echo "Building PISA CIFF"
cd pisa-ciff
cargo build --release
cd ../
```

```
echo "Building Anserini"
cd anserini
mvn clean package appassembler:assemble
cd eval
tar xvfz trec_eval.9.0.4.tar.gz && cd trec_eval.9.0.4 && make
cd ../../../
```

```
echo "Building Anserini-CIFF"
cd anserini-ciff
mvn clean package appassembler:assemble
cd ../
```

End to End Runs
---------------
If you are interested in generating an end-to-end run, simply run the
following scripts in order.
```
./prepare-data.sh
./build-indexes.sh
./search-effectiveness.sh
./search-efficiency.sh
```

Collections
-----------
We have four collections in jsonl format. They are the original MS-Marco corpus
tokenized by BERT, and the DeepCT version (same tokenization process). For each
index, both an 'unpruned' and a 'pruned' version are generated - the pruned
versions remove all postings which DeepCT weighted to zero. 

You can download and prepare these collections with the `prepare-data.sh`
script. For step-by-step instructions, check out the [data preparation](https://github.com/JMMackenzie/term-weighting-efficiency/blob/master/data-preparation.md) 
guide.

Building the Indexes
--------------------
Assuming you have built the tools above, you can run the `build-indexes.sh`
tool which will automatically build all of the required data. If you would
prefer to do it yourself, refer to the [step-by-step indexing](https://github.com/JMMackenzie/term-weighting-efficiency/blob/master/indexing.md) 
guide.


Running Effectiveness Experiments
---------------------------------
The `search-effectiveness.sh` script will execute all runs and output their
effectiveness values. The runs can be located in `evaluate/run-files` and
a summary of the effectiveness scores can be gathered as follows:
```
cd evaluate
./evaluate-runs.sh
```

Running Efficiency Experiments
------------------------------
The `search-efficiency.sh` script will run the timing experiments and output
per-query efficiency values.
After the `search-efficiency.sh` script has been executed, the results can be
found in `timings/all.tsv` as a tab-seperated file.

To get a summary of these timings:
```
cd timings
python3 tools/get-summary.py all.tsv
```
