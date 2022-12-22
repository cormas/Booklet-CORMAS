## Building a Cormas Model from Scratch: the stupid model
	instanceVariableNames: ''
	classVariableNames: ''
	package: 'Cormas-Model-myStupid'
	instanceVariableNames: 'agsize'
	classVariableNames: ''
	package: 'Cormas-Model-myStupid'
  instanceVariableNames: 'theCMStupidCells theCMStupidAgents numberOfStupidAgent'
  classVariableNames: ''
  package: 'Cormas-Model-myStupid'
	^ Color green.
	"move to an empty cell... if not don't move"

	self randomWalkConstrainedBy: [
			:c | c noOccupant
		].

	^ Color blue

	self move.
	^ numberOfStupidAgent ifNil: [ numberOfStupidAgent := 10 ]

	self createGridX: 100 Y: 100 neighbourhood: 4 closed: false.
	self setRandomlyLocatedAgents: CMStupidAgent n: self numberOfStupidAgent.
	| aModel |
	"self setActiveProbes: OrderedCollection new."
	aModel := self initialize new initSimulation.
	(CMSimulationGrid new
		on: aModel
		withCells: aModel theCMStupidCells
		withSituatedEntities: aModel theCMStupidAgents) runAndVisualizeWithMenus

	^ defaultInit ifNil: [ defaultInit := #initWithProgrammableScenario ]

	defaultInit := aSelector