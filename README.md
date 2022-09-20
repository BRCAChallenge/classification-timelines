# Summary 

Many genetic variants are classified, but many more are designated as variants of uncertain significance (VUS). Patient data may provide sufficient evidence to classify VUS. Understanding how long it would take to accumulate sufficient patient data to classify VUS can inform many important decisions such as data sharing, disease management, and functional assay development.

Our software models accumulation of clinical data and their impact on variant interpretation to illustrate the time and probability for variants to be classified when clinical laboratories share evidence, when they silo evidence, and when they share only variant interpretations. 


# Clone the repo and install dependencies
1. Clone this github repository to your local system. 

```console
$ git clone https://github.com/BRCAChallenge/classification-timelines 
```

2. Change directory to the `classification-timelines` directory. 

```console
$ cd classification-timelines 
```

3. Install dependendencies in your Python 3.x environment (Python 2.x is not supported).

```console
$ pip install -r requirements.txt
```

# Edit model constants in the `conf.json` file according to your experiment. 
All the configuration information for a simulation is stored in a JSON file. 

## Update the sequencing center sizes and testing rates 

You can configure the `small`, `medium`, and `large` initial sizes and annual testing rates for your experiment by editing the following JSON code block:

```console
		"smallInitialSize": 15000,
		"smallTestsPerYear": 3000,
		"mediumInitialSize": 150000,
		"mediumTestsPerYear": 30000,
		"largeInitialSize": 1000000,
		"largeTestsPerYear": 450000,
```


## Update the evidence category frequencies for pathogenic observations.

You can configure the evidence category frequencies for the pathogenic observations by editing the `low`, `med`, and `hi` values in the following JSON code block.  Note that the `med` value is used for the simulations, and the `low` and `hi` values are used only for the sensitivity analysis.

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


## Edit the evidence category frequencies for benign observations

You can configure the evidence category frequencies for the benign observations by editing the `low`, `med`, and `hi` values in the following JSON code block.  Note that the `med` value is used for the simulations, and the `low` and `hi` values are used only for the sensitivity analysis.

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

## Evidence likelihoods

The likelihoods of ACMG/AMP evidence strength for benign (`B`) and pathogenic (`P`) observations are defined as "strong" (e.g. `PS`, `BS`), "moderate" (e.g. `PM`), "supporting" (e.g. `PP`, `BP`), and "standalone" (not available for clinical data).  These likelihoods match Tavtigian's Bayesian framework likelihoods which were shown to be equivalent to the ACMG/AMP evidence strengths.  It's not recommended to change these values.

```console
    		"PS": 18.7,
    		"PM": 4.3,
    		"PP": 2.08,
    		"BS": 0.053475935828877004,
    		"BP": 0.4807692307692307,

``` 

## Pathogenic selection factor

Healthy people from healthy families are underrepresented in many forms of genetic testing. Accordingly, patients with pathogenic variants are observed (or ascertained) more often than those with benign variants, and the forms of evidence that support a pathogenic interpretation accumulate more quickly. How much more likely a person is to present pathogenic evidence than benign evidence is captured in our model as a configurable real-valued constant.

```console
    		"PSF": 2.0,

```

## Evidence thresholds

The thresholds configured in the repository's `conf.json` file are shown below.  These match Tavtigian's Bayesian framework thresholds which were shown to be equivalent to the ACMG/AMP evidence thresholds.  It's not recommended to change these values.

```console
		"benignThreshold": -3,
		"likelyBenignThreshold": -1.256958152560932,
		"neutralThreshold": 0,
		"likelyPathogenicThreshold": 1.256958152560932,
		"pathogenicThreshold": 2.0
```

# Configure an experiment

To configure an experimental run, update the following parameters.  
* `name`:		name to give your experiment
* `nSmall`:		number of "small" sequencing centers participating in experiment
* `nMedium`:		number of "medium" sequencing centers participating in experiment
* `nLarge`:		number of "large" sequencing centers participating in experiment
* `numVariants`:	number of variants to simultaneously experiment
* `frequency`:		allele frequency for variant in experiment
* `years`:		number of years over which to run the simulation
* `seed`:		integer seed for random number generation
* `numThreads`:		number of CPU threads to allocate to experiment


```console
	"simulation": {
		"name": "mySim",
		"nSmall": 10,
		"nMedium": 7,
		"nLarge": 3,
		"numVariants": 1000,
		"frequency": 1e-5,
		"years": 5,
		"seed": 18,
		"numThreads": 2
	}

``` 


# Run an experiment


