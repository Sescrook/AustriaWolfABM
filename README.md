Overview

1.1 Purpose
The purpose of the model is to simulate wolf movement in Austria. Once movement patterns have been established, the model is intended to assess the potential of different scenarios in resulting in wolf recolonization via pack formation.

1.2 Entities, state variables and scales
	The model contains three entities: individual wolf agents, packs formed by multiple wolf agents, and the environment over which processes operate. Wolf agents are described by the following state variables: their location (x and y coordinates), whether they are alive or dead, their sex, whether they have mated, and whether they have formed a pack.
	Individual wolf agents operate over an environment consisting of a rudimentary habitat suitability model. The suitability pixels are placed in one of three categories (0, 1, or 2). A suitability of 0 denotes unsuitable habitat, made up of urban areas, water, glaciers, and exposed rock/cliff. A suitability of 1 denotes marginal habitat, through which a dispersing wolf might travel, but which does not make up ideal core wolf habitat. This suitability class is predominantly made up of agriculture. A suitability class of 2 denotes good wolf habitat, which a wolf would likely use for transit or to constitute its core habitat. This class is predominantly made up of forest land cover. Each suitability pixel is 100 meter x 100 meter.
	Packs can also be considered entities, and are formed as the outcome of a number of different decision and proximity-based rules for wolf agents. A pack is created as a two dimensional area in the “packs” raster. The spatial extent of a pack depends on the pack size input parameter.
	The spatial extent of the model includes an area slightly larger than the boundaries of Austria (approximately 600 x 350 km). The habitat suitability raster was created to include a 30km buffer of the national boundary. Individual wolves, however, are able to travel outside this area (though there are rules introduced that make it difficult for them to exist “alive” for too long outside of the suitability raster, simulating their migration away from the study area).
	Each model tick, or step, is made to represent two weeks of time. For the movement submodel, there are sub-ticks in which the wolf agents decide on movement destination on an hourly basis. The model is intended to be run over moderate temporal scales (from five to ten years). One hundred and thirty ticks makes up five years, while two hundred and sixty ticks represent ten years.
	
1.3 Process Overview and Scheduling
The model proceeds in biweekly (once every two weeks) time steps as illustrated in Figure 4. There is an initialization step, in which rasters are loaded into the model and starting wolves are placed according to the number specified in model parameters, prior to the running of any steps. In each of the subsequent model steps, a number of actions take place for each wolf in order of the wolf’s ID number.
	First, new wolves appear in the model environment. Then, wolf movement submodel runs (including hourly substeps). Next, wolves search for a mate, and potentially mate (depending on input parameter values). If there are wolves who have mated, they may establish a pack if there is sufficient good quality habitat (again, determined by user selected specified input parameters). Finally, wolves die by either leaving the study area or based on mortality probabilities.

2. Design Concepts

2.1 Basic Principles
The basic principle of the Austria Wolf Model is to give information to assess whether qualitatively observed patterns of wolf dispersal, presence, and behavior in Austria can be used to estimate the likelihood that permanent wolf populations may be formed in the country (as wolf packs).

2.2 Emergence
The Austria Wolf Model includes the concept of emergence by capturing the system level phenomenon of wolf populations and pack formation over a large area through the decisions and movement of individuals. Formation of packs, and the number of packs formed are wholly results of rule sets that arise from individual decision-making and stochasticity.

2.3 Objectives
The rule sets of the model are designed to make the establishment of a pack the ultimate objective of wolf agents.

2.4 Sensing
This model includes sensing in three main ways. First, the wolf agents are able to sense their environment. As part of the move method, they partake in a biased random walk by checking the environment pixel nearby where they may move to, and deciding whether or not to move there based on probabilities. Secondly, male wolves look around themselves at every time step in order to find potential mates, sensing the presence of female wolves within a specified radius. Finally, wolf agents that have found a potential mate once again sense their environment, assessing their surroundings and tabulating how many nearby pixels are of suitable habitat for pack formation. 

2.5 Interaction
Wolf agents interact with each other at two steps. First, wolves of opposite sex are considered potential mates based on proximity and may, based on specified probabilities, mark each other as a mate. Then, based on their assessment of surrounding habitat, they may further form a pack by changing each other’s establishTerritory field.

2.6 Stochasticity
The model contains a substantial amount of stochasticity in order to find various possibilities inherent in different scenarios. The distances wolves travel in a given two week period are the sum of distances randomly drawn hourly from a normal distribution based on hypothesized wolf movement over longer time periods. The decision of whether or not to move to different habitat types is based on drawing from random distributions. Death each step is determined by a certain probability specified in model parameters. Even the decision to mate upon meeting a wolf of the opposite sex can be assigned a probability.

2.7 Collectives
The model does not explicitly model collectives, but the formation of packs can be seen as the beginning of the formation of a collective.

2.8 Observation
During model testing steps, the spatial distribution of wolves and packs at each time step was observed. For analysis over several model runs, the number of packs formed and the average number of wolves alive at each time step over the five year period were compiled to compare scenarios. Furthermore, all of the locations wolves ended up at the end of ticks and all of the packs formed over several model runs were written to separate raster files to get a qualitative overview of density and an idea where these phenomenon were occurring. For some model runs, pack areas were output to see where are likely locations of pack formation (writing of pack areas to raster is computationally intensive and therefore not run for all models).

3.1 Initialization
Prior to the first time step, the environment is loaded into the model, and “starting wolves” are placed on the landscape, representing the wolves that are already in Austria. The number of these wolves is selected by the user prior to the model run. Each starting wolf is randomly placed in “good” habitat within the national borders of Austria.

3.2 Input
The environment is based on an external data file. Corrine land cover classification data (2006) at 100m spatial resolution was adapted to create a habitat suitability model for Austria (see section METHODS.X ) and a 30 km buffer around Austria (European Environment Agency).

3.3 Submodels
Each time step, the following submodels are run: appear, move, mate, establish pack, and die (Figure 4). 

 
Figure 4 – Model Flow Chart

The appear submodel runs at two levels. At the model level, at each time step, the annual rate of new wolf parameter is converted to a biweekly probability using the following equation:

P = 1-e-λt 

A random number between 0 and 1 is drawn at each time step, and if that number is smaller than the biweekly probability, a wolf is created (nextWolf, which is a variable that designates which wolf is next in line to be activated and brought into the model). The model level code indicates that a new wolf will be created, while the wolfLevel submodel activates the individual wolf (nextWolf). The newly activated wolf has its “alive” variable set to 1 (alive), is randomly placed within the start box (near the Austria, Italy, Slovenia border; see Figure XX), and the sex is set using a coin toss (random integer is generated with either 0 or 1).
	The move submodel is conducted by all living wolves (self.alive = 1) that have not established a pack (self.establishPack = 0). In it, wolves move hourly based on average movement distances and surrounding habitat types. First, the daily average and standard deviations for movement (parameters) are converted to hourly distances. For each wolf, 336 hourly movement decisions take place at every time step. At each of the 336 hours, X and Y distances for movement are randomly drawn from a normal distribution of hourly mean distance and hourly standard deviation variables. Direction (whether the X and Y movements are in the positive or negative directions) is chosen based on random binary draw.
	While these potential movement distances and directions are established here, the wolf only moves based on habitat suitability and some randomness, undertaking a movement trajectory that might be considered a highly biased random walk. At each hourly time step, the wolf agent checks the suitability of the proposed cell found above. Using random numbers, the following rules are imposed: if the suitability is poor, the wolf agent has a 5% chance of moving to it, if the suitability is marginal, the wolf agent has a 75% chance of moving to it, if the suitability is good, the wolf agent has a 95% chance of moving to it, and if the cell is out of bounds, the wolf agent has a 70% chance of moving to it. If the wolf moves out of bounds, there is a chance it leaves the study area completely (see the die sub model).
	The mate sub model allows wolf agents to detect other wolf agents of the opposite sex within a specified radium (model parameter). The sub model is run for all “male” wolf agents that are “alive” and have not already found a mate. Each male wolf agent calculates distance to all female wolves according to:

d= √((x_2-x_1 )^2+(y_2-y_1 )^2 )
	
If the distance between a male wolf agent and a female wolf agent is less than the parameter value for minimum distance for detecting a mate, a random number draw takes place. If the parameter for mating probability is less than the random number draw, the wolf agents are marked as potential mates and their foundMate variable is changed to 1.
	The Establish Pack sub model is conducted by all male wolf agents that have found a mate and have not already established a pack. In this submodel, the wolf agent checks surrounding territory for suitability, and if found to have the requisite amount of suitable territory, establishes a pack. First, bounds of the potential pack area are calculated by converting wolf location in map coordinates to wolf location on the raster grid. Minimum and maximum bounding pixels define the potential pack box based on packArea parameter (km2). The wolf checks every pixel within the potential pack box, summing pixels of good habitat and total pixels, ultimately coming up with a measure of the percentage of good habitat. If the percentage of good habitat is greater than the percentGoodHabitatForPack parameter, the wolf establishes a pack. In some model runs, the wolf agent then outputs the location of the pack to raster by writing new pixel values to each pixel within the pack box (this, however, is computationally taxing).
	The die sub model is run by each living wolf agent. The annual mortality rate is converted to a biweekly mortality rate (using equation 1), and a random decimal between 0 and 1 is drawn for each living wolf agent. If the random number is less than the mortality probability, the wolf agent is no longer considered alive (and therefore no longer carries out any of the submodels listed above). In addition to death due to random probabilities, out of bounds wolves (found to finish the turn out of the study area during the move submodel) have a 25% chance of leaving the study area (thus no longer being considered alive). This means, however, that they have a 75% chance to remain alive, return to the study area, and persist for future time steps even if out of the study area.

4.1 Model parameters and calibration
	Because wolf recolonization is poorly understood in Austria, and only fragmented data exist regarding wolf movement and presence in the country, the model was primarily calibrated using a range of literature values. Values for specific wolf behaviors derived from the literature were related to average movement distance, average mortality, the distance wolves can detect potential mates, the area required for a pack, and the number of starting wolves in Austria (Figure 1). Parameters qualitatively derived from general literature review included the rate of new wolf appearance, the amount of good habitat required within the pack area for a pack to form, and mating probability.


Three different models were run ten times each to reflect the range of literature-derived parameters (Figure XX). These models were meant to demonstrate the range of possible situations that might reflect the real-world situation. The first model (M1) used the values which would likely lead to low probabilities of wolf recolonization. The second model (M2) used the values which would likely lead to high probabilities of wolf recolonization. The third model (M3) used intermediate values which would lead to moderate probabilities of wolf recolonization.
