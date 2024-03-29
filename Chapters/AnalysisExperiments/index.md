## Analysis and experiments Cormas models


### Analysis and experiments with R


### OpenMole massive parallel analyzes and exploration


OpenMOLEhas been developed since 2008 as a free and open-source platform. It offers tools to run, explore, diagnose and optimize your numerical model, taking advantage of distributed computing environments. With OpenMOLE you can explore your already developed model, in any language \(Java, Binary exe, NetLogo, R, SciLab, Python, C++...\).

OpenMOLE comes with a graphical user interface \(GUI\) to write scripts around your model. These scripts will use OpenMOLE methods to explore your model and distribute its executions on your laptop as on High Performing Computing \(HPC\) environments, with only a few lines of code!

OpenMOLE is the tool you need if you want to carry out Real sensitivity analysis, calibration on mono or multi-criteria, Pattern diversity research in model dynamics, custom design of experiments, data processing.

#### The basis for dialogue


OpenMOLE work with docker. Regarding figure *@OMArch@*, OpenMOLE is downloading a docker image of CORMAS and at the same time, it generates a set of parameter files in json. The CORMAS docker will be embedded in an archive sent to you dedicated computer \(we will call it clusters\). Each archive will be a job and those jobs will be assigned to the cluster scheduler. They will wait in the queue... and when they are achieved the result will be sent to you openMOLE instance.

![OpenMole and CORMAS architecture](figures/Cormas_openMole.png width=80%&label=OMArch)

#### Script on openMOLE side


We will take a look at a basic openMOLE script step by step based on a Cormas toy model : FireAutomata.

The first step is to map our input and output variables according to the model variable in order to allow communication between OpenMOLE variable and CORMAS variable.
This model has 3 input variables : the initial number of fires, the number of firemen and the percentage of trees \(some cell are bare sol\). And we are interested by 1 output variable : the number of trees not burned at the end of the simulation.

In openMOLE variable declarationslook like that :

```
val seed = Val[Int] // defined a random seed

val numberOfFires = Val[Int] //input variable
val numberOfFiremen = Val[Int] //input variable
val percentageOfTrees = Val[Double] //input variable
val dimensionMin = Val[Int]  //input variable : grid dimensions X
val dimensionMax = Val[Int]  //input variable : grid dimensions Y
val nbTrees = Val[Int] //output variable
```


The link between openMOLE and CORMAS is made by the `CORMASTask()`. In this task you passe different input and output. And you call the specific methods defined to run a model \(we will describe this method below\).
If you want to fix some variable \(and exclude them from the sampling method that we describe below\) you can do that as follows.

```
// The CORMASTask take in parameters the class and method able to
// launch our simulation. In OUr case using the Cormas-Model-FireAutomata
// we run a method build for OpenMole. You can take a look.
// set() allow you to pass some other thing to your task. You can pass :
// * your model as a file.st
// * inputs from OpenMole Variable
// * outputs as an array
// * defined parameters how doesn't change between simulation.

val model = CORMASTask("CMFireAutomataModel simuOpenMole") set (
  //resources += workDirectory / "Cormas-Model-FireAutomata.st",
  inputs += seed,
  cormasInputs += numberOfFires,
  cormasInputs += numberOfFiremen,
  cormasInputs += percentageOfTrees,
  cormasInputs += dimensionMin,
  cormasInputs += dimensionMax,
  cormasOutputs += nbTrees,

  outputs += (seed, numberOfFires, numberOfFiremen, percentageOfTrees, dimensionMin, dimensionMax),

  numberOfFires := 3,
  numberOfFiremen := 10,
  percentageOfTrees := 0.65,
  dimensionMin := 60,
  dimensionMax := 80
)
```


You can worry about the calculation environment. OpenMOLE allow dealing transparently with local CPU as grided computer.

For example, with `LocalEnvironment(2)` you will use your local CPU here with 2 threads
```
val env = LocalEnvironment(2)
```


while with `SGEEnvironment()` you can deal with SGE cluster scheduler .

```
val env =
  SGEEnvironment(
    "login",
    "cluster.url",
    queue = "normal.q",
    workDirectory = "/homedir/user/work"
  )
```


A lot of other environments are supported so doesn't esitate to tack a look on documentation.

As for the environment, openMOLE embedded a lot of smart sampling and exploration methods. For this example we will use a basic one : `DirectSampling()`.
In this script it's sampling method how will prepare all the jobs archives. So you specify the hook \(a wait to take output from the model\), the sampling \(based here on `numberOfFires` between 1 to 10 fire\), and the number of replication with `take 100`.

```
// With the DirectSampling() method you define an easy wait to generate
// a sampling for numberOfFires between 1 to 10.
DirectSampling(
  evaluation = model on env by 50 hook CSVHook(workDirectory / "results_by50_sge.csv"),
  sampling = (numberOfFires in (1 to 10)) x
  (seed in (UniformDistribution[Int]() take 100))
)
```


Finally, it looks like that :

```
// OpenMole Variable definition.
// Those variables will be populated in openMole
// and send in JSON to Pharo/Cormas

val seed = Val[Int]

val numberOfFires = Val[Int]
val numberOfFiremen = Val[Int]
val percentageOfTrees = Val[Double]
val dimensionMin = Val[Int]
val dimensionMax = Val[Int]
val nbTrees = Val[Int]

// The CORMASTask take in parameters the class and method able to
// launch our simulation. In OUr case using the Cormas-Model-FireAutomata
// we run a methods build for OpenMole. You can take a look.
// set() allow you to pass some other thing to your task. You can pass :
// * your model as a file.st
// * inputs from OpenMole Variable
// * outputs as an array
// * defined parameters how doesn't change between simulation.

val model = CORMASTask("CMFireAutomataModel simuOpenMole") set (
  //resources += workDirectory / "Cormas-Model-FireAutomata.st",
  inputs += seed,
  cormasInputs += numberOfFires,
  cormasInputs += numberOfFiremen,
  cormasInputs += percentageOfTrees,
  cormasInputs += dimensionMin,
  cormasInputs += dimensionMax,
  cormasOutputs += nbTrees,

  outputs += (seed, numberOfFires, numberOfFiremen, percentageOfTrees, dimensionMin, dimensionMax),

  numberOfFires := 3,
  numberOfFiremen := 10,
  percentageOfTrees := 0.65,
  dimensionMin := 60,
  dimensionMax := 80
)

//val env = LocalEnvironment(2)

val env =
  SGEEnvironment(
    "login",
    "cluster.url",
    queue = "normal.q",
    workDirectory = "/homedir/user/work"
  )

// With the DirectSampling() method you define an easy wait to generate
// a sampling for numberOfFires between 1 to 10.
DirectSampling(
  evaluation = model on env by 50 hook CSVHook(workDirectory / "results_by50_sge.csv"),
  sampling = (numberOfFires in (1 to 10)) x
  (seed in (UniformDistribution[Int]() take 100))
)
```


Once you have a better idea about how openMOLE and CORMAS will talk each other we will take a look at CORMAS model and methods.

#### Some change in our models


To run your model in OpenMOLE you need to introduce a method that will serve for communication between the model itself and OpenMOLE \(step 2 fig. *@OMArch@*\).
You must introduce two methods: `CMOpenMoleExchange loadJSONFile: `input.json' ` \(to read the configuration file produced by openMOLE\) and `CMOpenMoleExchange saveJSONFile: (CMOpenMoleExchange lastDataOfModel: aModel)` \(to save the results of the probes that interest you in an output file `json`.

```
simuOpenMole
	"Example used in OpenMole (https://openmole.org/)"
    | conf aModel |
	 "self createInJSONFile." "OpenMole usually generate this file"
    conf := CMOpenMoleExchange loadJSONFile: 'input.json'.
    aModel := self newWithProgrammableScenario
        numberOfFires: (conf at: #numberOfFires) ;
        numberOfFiremen: (conf at: #numberOfFiremen);
        percentageOfTrees: (conf at: #percentageOfTrees);
        dimensions: (conf at: #dimensionMin) -> (conf at: #dimensionMax);
        activeControl: #step:;
        initSimulation.
    aModel simManager
        initializeSimulation;
        finalTime: 100;
        runSimulation.
    CMOpenMoleExchange saveJSONFile: (self lastValuesOfVariables: (aModel data))
```




#### Benchmarking on fireman


On “Fireman” model under:


| 100 replications | \~ |
| --- | --- |
| Desktop computer | 6 min |
| CIRAD cluster | 22 min |


| 1000 replications | \~ |
| --- | --- |
| Desktop computer | 60 min |
| CIRAD cluster | 37 min |


| 10000 replications | \~ |
| --- | --- |
| Desktop computer | 10h |
| CIRAD cluster | 1h38 |