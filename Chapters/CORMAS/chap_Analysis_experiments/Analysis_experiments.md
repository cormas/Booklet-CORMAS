## Analysis and experiments Cormas models
val seed = Val[Int] // defined a random seed

val numberOfFires = Val[Int] //input variable
val numberOfFiremen = Val[Int] //input variable
val percentageOfTrees = Val[Double] //input variable
val dimensionMin = Val[Int]  //input variable : grid dimensions X
val dimensionMax = Val[Int]  //input variable : grid dimensions Y
val nbTrees = Val[Int] //output variable
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
  SGEEnvironment(
    "login",
    "cluster.url",
    queue = "normal.q",
    workDirectory = "/homedir/user/work"
  )
// a sampling for numberOfFires between 1 to 10.
DirectSampling(
  evaluation = model on env by 50 hook CSVHook(workDirectory / "results_by50_sge.csv"),
  sampling = (numberOfFires in (1 to 10)) x
  (seed in (UniformDistribution[Int]() take 100))
)
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