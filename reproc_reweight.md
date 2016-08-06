## Instructions for using driver to reweight a simulation

This is a short outline of things to check.

You will need a trajectory, tpr file (to modify the trajectory), and a COLVAR or HILLS file

1) Reweighting in plumed can only be for 1 or 2 CV's. Anything greater will lead
to a segmentation fault. Make sure you are using PLUMED 2.2 or later.

2) Decide which file you would like to use for input (COLVAR or HILLS)
- Adjust your trajectory to match the pace to which your selected file was written.
Ex: IF you originally wrote to your trajectory every 100 steps, but wrote to your COLVAR every 1000 steps, then you need modify your trajectory to only have every 10 steps now to match up the timing.
- If you chose to use your HILLS file, copy the first line of data, paste one line above and change the time to 0. This is because you need an initial coordinate to start (irrelevant what it is) or else you will skip your first fill.

3) Create plumed.dat file (example given below)
- use READ function to read in your CV(s) from the selected input file
- use IGNORE_FORCES and IGNORE_TIME as this is entirely post-processing where your CV values are fixed

- Ensure you have parameters for GRID (MIN, MAX, BIN) and Bias factor or else REWEIGHTING_NGRID will not work. Make sure your REWEIGHTING grid size is as large or larger than GRID

- Make sure you adjust pace to match the new time step of your adjusted trajectory.If you are using HILLS as an input your PACE=1 because each frame is a frame you previously deposited a hill.


4) Run with driver: `plumed driver --plumed plumed.dat --mf_xtc traj.xtc.`

Your new COLVAR and HILLS files should match your original ones, except the time will be different, and your COLVAR will have rbias and rct as well. You should check to make sure your CV values and bias values are preserved to ensure everything is the same.


Example plumed.dat file (alanine dipeptide)

phi: READ FILE=HILLS_old VALUES=phi IGNORE_FORCES IGNORE_TIME
psi: READ FILE=HILLS_old VALUES=psi IGNORE_FORCES IGNORE_TIME

METAD ARG=phi,psi SIGMA=0.2,0.2 FILE=HILLS HEIGHT=1.2552 PACE=1 BIASFACTOR=9 TEMP=300 LABEL=metad GRID_MIN=-pi,-pi GRID_MAX=pi,pi GRID_SPACING=0.1,0.1 REWEIGHTING_NGRID=100,100 REWEIGHTING_NHILLS=1


PRINT ARG=* STRIDE=1 FILE=COLVAR


More questions checkout PLUMED Manual
REWEIGHTING [http://plumed.github.io/doc-master/user-doc/html/_m_e_t_a_d.html]
DRIVER[http://plumed.github.io/doc-master/user-doc/html/driver.html]
