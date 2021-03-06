**************************************************
*                                                *
*  SIRE: Software for IBS and Radiation Effects  *
*                                                *
**************************************************

Based on MOCAC

Developed by A. Vivoli (2010)
Revised by F. Antoniou (2014)
Revised by Stefania Papadopoulou (2016)


***********************************
*                                 *
* HOW TO COMPILE AND RUN THE CODE *
*                                 *
***********************************


1. Compile the code (connected to lxplus, because it may not work on my pc):			(sometimes it needs to: chmod +x file)
  g++ lhcsire.c -o code
  
2. Run the code :
  ./code twissfile.tfs paramsfile.dat name 	name for the output files
  ./code twissfile.tfs paramsfile.dat name distribution.txt 	if the distribution is given as input, if not it assums gaussian

or for submitting jobs (ex. if scanning):
  ./ScanNcellsOptimization.sh (connected to lxplus) (inside which there is the batchsubmit.sh). 

At least 3 arguments are required. If less then the code gives an error. If 4 arguments then the 4rth one should be the input distribution file

  arg1 --> The madx twiss file. Columns has to follow the ordering: name,s1,len1,betx1,alphax1,mux1,bety1,alphay1,muy1,dx1,dpx1,dy1,dpy1

  arg2 --> The input parameters file. 
    TEMPO: The full time of simulation	
    nturnstostudy: total number of turns 
    NIBSruns: Number of timesteps you devide the TEMPO to calculate IBS
    oneturn: 1 for 1-turn calculation, 0 for multi-turn
    TIMEINJ: -> time step
    fastrun: 1 if interpolation is used every TIMEINJ, NIBSruns determines how many turns are skipped for the IBS calculation. So be careful to have a sufficient NIBSruns for a specific time.
    continuation: 1 so that if a job is killed it continues from where it stopped
    epsx: Starting horizontal emittance (the geometrical!)
    epsz: Starting vertical emittance (the geometrical!)
    delta: Starting energy spread
    deltas: Starting bunch length. It should be given in [meter] the 1 sigma, then for 1ns blength (4sigma) in the params file we should put blength=(1ns/4)*clight. whenever changing blength->change also en.spread
    dtimex: horizontal damping time
    dtimez: vertical damping time
    dtimes: longitudinal damping time
    eqx: Equilibrium horizontal emittance
    eqz: Equilibrium vertical emittance
    eqdelta: Equilibrium energy spread
    eqdeltas: Equilibrium bunch length
    flag_rec: 0 if full lattice, 1 to use smaller number of lattice points
    damping: if 0 the radiation damping not taken into account 	
    IBSflag: if 0 no IBS calculations
    q_ex: if 0 the quantum excitation not taken into account 			
    momentum: Beam momentum [MeV]
    massparticle: [MeV]
    chargeparticle: The charge of the particle (1 for e-)
    numbunch: bunch population	
    numpart: Number of macroparticles		
    ncellx: # cells in the horizontal plane
    ncellz: # cells in the vertical plane
    ncells: # cells in the longitudinal plane		
    ncollisions: # collisions per particle (close encounters)
    convsteadystate: Track until convergeance to steady-state not in full TEMPO time
    checktime: 0 if you want to see the TEMPO, 1 if it sees the turns instead of the TEMPO.

in sire code:
	precision: tells you how tha reccurences work. Elements of the lattice with twiss functions differing of less than prec.% are considered equal.
	The closer it is to 1->more reccurences->shorter lattice->less time-> but also less accurancy. The closer it is to 0->less recurences->higher accurancy.
	
	renormbinningtrans=1,renormbinningall=1 sees ncellx and ncellz as f(ncells) using the mppercell
	mppercell=5 is the #mp/cell


3. Output files using the arg3 for the naming of the output files:
  RES_TWISS_arg3.txt --> The new twiss file after recurrences
  RES_EMITTANCE_arg3.txt --> The emittance at each point of the lattice after the IBS kicks (for 1-turn calculations only). Four columns (s, exm, ezm, esm)
  RES_Growth_Rates_arg3.txt --> File with emittances for each time step! The Growth rates are the zeroed columns, so they are not saved. 
				    In this file, L{1}=timesteps (so the NIBSruns), L{2},L{3}=the emittances and energyspread=sqrt(L{4}/2).
  RES_DISTRIB_arg3.txt --> The distribution file, that has a length=#mp.
  we can also ask for the output distribution.
  

*****************************
*                           *
* MORE COMMENTS AND DETAILS *
*                           *
*****************************

1. Structure of main: 
  a. Reads madx and input parameters files and generates the names of the output files
  b. if flag_rec then optimizes the number of points in the lattice to be used in order to speed up the simulation --> Generates the new twiss file
  c. Generates the distribution of macroparticles given the mean invariants values and random phases
  d. if the IBSflag=1 then apply the transformation to the momentum frame, then apply the IBS routine in each point of the lattice, transform back to the invariants frame
  e. If oneturn=1 the above calculation is done only once and the one-turn emittance evolution around the lattice is writen in an output file
  e. If oneturn=0 the tracking of the distribution evolution is followed for every element anf for each turn for a timestep TEMPO or until convergance. The mean growth times and mean emittances are calculated and written in an output file every one timestep TIMEINJ
  f. If damping=1 and q_ex=1 then the effects of synchrotron radiation damping and quantum excitation are taken into account.
  
2.  How the IBS routine works:
  a. Finds the limits of the distributions
  b. The distributions are splitted in cells (uniform split with respect to x, z, s)
  c. Puts the macroparticles in cells and calculates the number of macroparticles and the density of mp in each cell
  d. Applies the scattering kicks between the macroparticles in each cell (binary collisions) --> redestribution of phase space
  
3. How the fastrun is applied: 
  a. The IBS is applied in oneturn and the growth rate per particle is calculated
  b. The emittance per particle is recalculated based on the exponential IBS growth in a timestep TIMEINJ. 
  c. contunue tracking with the interpolated distribution

  Comment: The fastrun is applied in order to speed up the simulation. The user has to check if it gives valid results   
  
  
