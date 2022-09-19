# Summary 

Many genetic variants are classified, but many more are designated as variants of uncertain significance (VUS). Patient data may provide sufficient evidence to classify VUS. Understanding how long it would take to accumulate sufficient patient data to classify VUS can inform many important decisions such as data sharing, disease management, and functional assay development.

Our software models accumulation of clinical data and their impact on variant interpretation to illustrate the time and probability for variants to be classified when clinical laboratories share evidence, when they silo evidence, and when they share only variant interpretations. 


# Clone the repo.
1. Clone this github repository to your local system. 

```console
$ git clone https://github.com/BRCAChallenge/classification-timelines 
```

2. Change directory to the `classification-timelines` directory. 

```console
$ cd classification-timelines 
```

# Configure a simulation.
All the configuration information for a simulation is stored in a JSON file. 

1. You can configure the `small`, `medium`, and `large` initial sizes and annual testing rates for your experiment by editing the following JSON code block:

```console
		"smallInitialSize": 15000,
		"smallTestsPerYear": 3000,
		"mediumInitialSize": 150000,
		"mediumTestsPerYear": 30000,
		"largeInitialSize": 1000000,
		"largeTestsPerYear": 450000,
```


2. You can configure the evidence category frequencies for the pathogenic observations by editing the `low`, `med`, and `hi` values in the following JSON code block.  Note that the `med` value is used for the simulations, and the `low` and `hi` values are used only for the sensitivity analysis.

```console
		"p0": {
			"low": 0,
			"med": 0,
			"hi": 0
			},
		"p1_PM3": {
			"low": 0,
			"med": 0,
			"hi": 0
			},
    		"p2_PM6": {
			"low": 0.0014,
			"med": 0.007,
			"hi": 0.025
			},
    		"p3_BS2": {
			"low": 0,
			"med": 0,
			"hi": 0
			},
    		"p4_BP2": {
			"low": "0.001 * self.frequency",
			"med": "0.005 * self.frequency",
			"hi": "0.02 * self.frequency"
			},
    		"p5_BP5": {
			"low": 0.00002,
			"med": 0.0001,
			"hi": 0.0021505376
			},
    		"p6_PP1": {
			"low": 0.05,
			"med": 0.23,
			"hi": 0.67
			},
    		"p7_PS2": {
			"low": 0.0006,
			"med": 0.003,
			"hi": 0.02
			},
    		"p8_BS4": {
			"low": 0.0001,
			"med": 0.001,
			"hi": 0.17
			},

```


3. You can configure the evidence category frequencies for the benign observations by editing the `low`, `med`, and `hi` values in the following JSON code block.  Note that the `med` value is used for the simulations, and the `low` and `hi` values are used only for the sensitivity analysis.

```console
   		"b0": {
			"low": 0,
			"med": 0,
			"hi": 0
			},
    		"b1_PM3": {
			"low": 0,
			"med": 0,
			"hi": 0
			},
    		"b2_PM6": {
			"low": 0.0007,
			"med": 0.0035,
			"hi": 0.01
			},
    		"b3_BS2": {
			"low": 0,
			"med": 0,
			"hi": 0
			},
    		"b4_BP2": {
			"low": "self.frequency",
			"med": "self.frequency",
			"hi": "self.frequency"
			},
    		"b5_BP5": {
			"low": 0.038,
			"med": 0.099,
			"hi": 0.36
			},
		"b6_PP1": {
			"low": 0.005,
			"med": 0.01,
			"hi": 0.0625
			},
    		"b7_PS2": {
			"low": 0.0001,
			"med": 0.0015,
			"hi": 0.005
			},
    		"b8_BS4": {
			"low": 0.025,
			"med": 0.1,
			"hi": 0.4063
			},
```

## Permute the expression data and the metadata 
IMPORTANT: Run the following steps to permute the data and metadata so that the samples are in the same order in both the expression file and the metadata file.  The GAN training algorithm assumes the samples are in the same order, and if they are not, it will produce jibberish results.

1. Examine the first column of the original expression data and first row of the original meta data.  Note that they are not the same.
```console
$ sed -n '1,1 p' examples/data/expr.csv | awk -F, '{print $2}' 

$ sed -n '2,2 p' examples/data/meta.csv | awk -F, '{print $1}'
```

2. Run the `permuteSamples.py` script.  
* `-e` option specifies the expression data file 
* `-m` option specifies the metadata file
* `-mk` option specifies the metadata key 
* `-ek` option specifies the expression data key 

```console
$ python utils/permuteSamples.py -e examples/data/expr.csv -m examples/data/meta.csv -mk Sample -ek gene
```


3. Examine the first column of the permuted expression data and first row of the permuted meta data.  Note that they are the same.

```console
$ sed -n '1,1 p' examples/data/expr_permuted.csv | awk -F, '{print $2}' 

$ sed -n '2,2 p' examples/data/meta_permuted.csv | awk -F, '{print $1}'
```


## Reduce the dimensionality of the original data set. 
To reduce the number of rows (genes) in a data set, perform the following steps:

1. Run the `wc` command to determine the number of genes  

```console
$ wc -l examples/data/expr_permuted.csv 
```

2. Run the `reduceDim.py` script. The options are described below:
* `-n` option specifies the number of genes with the highest variance to keep
* `-d` option specifies the difference threshold between the highest and lowest expression level below which genes should be removed
* `-a` option specifies the percentage threshold (out of 100) of genes with zero expression above which genes should be removed
* `-e` option specifies the input expression file.

```console
$ python utils/reduceDim.py -e examples/data/expr_permuted.csv -n 25000 -d 10 -a 90
```

3. Run the `wc` command to determine number of genes after reduction. 
```console
$ wc -l examples/data/expr_permuted__reduced_25000_10_0.9.csv 
```

## Increase the number of technical replicates  
To increase the number of technical replicates, perform the following steps:

1. Determine the number of samples in the original data set.
```console
$ wc -l examples/data/meta_permuted.csv  
```

2. Run the `statistically_technical_replicate.py` script.
* `-e` specifies the input expression file
* `-m` specifies the input metadata file
* `-n` specifies the number of times more samples to create
* `-v` specifies the variance to use for the zero-mean gaussian sampling
* `-k` specifies the metadata key

```console
$ python utils/statistically_technical_replicate.py \
-e examples/data/expr_permuted__reduced_25000_10_0.9.csv \
-m examples/data/meta_permuted.csv \
-n 50 \
-v 10 \
-k 'Sample'
```

3. Determine the number of samples in the amplified data set.
```console
$ wc -l examples/data/meta_permuted__expanded_50_10.0.csv  
```

## Use a GAN to generate a fake data set
To create a synthetic gene expression data set, perform the following steps:

1. Create a meta JSON file to indicate which meta data are numerical and which are categorical.
```console
$ cat examples/data/meta.json 
```

2. Make an output directory
```console
$ mkdir /tmp/gan-out 
```


3. Run the `gen_fake_expr.py` script.  Note that this script will take several minutes to run. The number of epochs we define here as 10 is fewer than desired for the accuracy of the GAN, but for the sake of quickly getting results to examine, the value is set is intentionally low.  Set the number of epochs to 100 or higher to get more realistic, fake generated expression data.
* `-ie` is the input expression file
* `-im` is the input metadata file
* `-od` is the output directory
* `-umf` is the JSON file defining which categorical and numerical values to use
* `-e` is the number of epochs
* `-s` is the seed for the random number generator
* `-cd` is the checkpoint directory

```console
$ python GAN/gen_fake_expr.py \
-ie examples/data/expr_permuted__reduced_25000_10_0.9__expanded_50_10.0.csv  \
-im examples/data/meta_permuted__expanded_50_10.0.csv \
-od  /tmp/gan-out  \
-umf examples/data/meta.json \
-e 10 \
-s 23 \
-cd /tmp/gan-out
```

3. Examine the output
```console
$ ls -R /tmp/gan-out 
```

## Compare PCA plots 
To compare the PCA plots of original expression data to replicated data and generated fake data, run the following steps. 

1. Create a meta.json file mapping meta data as either numerical or categorical

```console
$ cat examples/data/meta.json 
```

2. Run the `plotters.py` script.
* `-ie` defines the input expression data file
* `-im` defines the input metadata file
* `-umf` defines the metadata JSON file which maps parameters as numerical or categorical
* `-od` defines the output directory

```console
$ python utils/plotters.py \
-ie examples/data/expr_permuted.csv  \
-im examples/data/meta_permuted.csv \
-umf examples/data/meta.json \
-od /tmp/gan-out
```

3. Examine the output to compare PCA plots
 
```console
$ open /tmp/gan-out/libPrep*.png
$ open /tmp/gan-out/mission*.png
```

## NOTES
We've found the following values to be optimal when generating RNA-seq data on NASA GLDS samples:

```console
 -ld 64 -bs 16 -nl 2 -hd 256 -lr 5e-04 -nb 5
```
