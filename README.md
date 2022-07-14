# SIREn (*S*ubunit *I*nference from *R*eal-space *En*sembles)
Automated detection of structural blocks in cryo-EM volume ensembles 
  
## Literature
     
   
## Installing  
First create a conda environment for SIREn, as shown below:
```
conda create --name siren python=3.9
conda activate siren
conda install pandas jupyterlab matplotlib scipy
pip install networkx
```

The scripts also require a cryoDRGN installation; instructions for downloading cryoDRGN can be found [here](https://github.com/zhonge/cryodrgn). Note that you do not need to install the cryoDRGN dependencies. SIREn is currently compatible with cryoDRGN v0.3.2. 

It is also recommended that you install ChimeraX (install instructions can be found [here](https://www.cgl.ucsf.edu/chimera/download.html)) in order to be able to visualize the structural blocks resulting from your SIREn analysis.

To install the SIREn code, simply git clone the source code:
```
git clone https://github.com/lkinman/siren.git
cd siren
python setup.py install
```

## Inputs
To run SIREn, you need a large ensemble of 3D density maps. These 3D density maps can come from any upstream processing software, but must be the same boxsize and aligned to a common reference frame. Additionally, the volumes must be downsampled -- we typically use a boxsize of 64, and other boxsizes have not been rigorously tested. Large boxsizes will lead to combinatorial explosions in required computational time/power. 

You will also need to supply a threshold for binarization of the maps. If there is not a common threshold that works well across all input maps, it may be appropriate to amplitude-scale the maps before running SIREn. 

Lastly, we note that the ensemble must be large. We typically use at least 500 volumes. This approach relies on the statistical power of sampling large numbers of volumes, so using smaller ensembles will result in decreased sensitivity.

## Outputs
SIREn outputs a dictionary (saved as a .pkl file that can be loaded into the Jupyter notebook generated by ```expand_communities.py```), where key-value pairs consist of each block's identifying number and constituent voxels. Additionally, the scripts automatically write out an individual .mrc file for each block. These .mrc files can be visualized in ChimeraX 

## Using  
This software comprises 3 scripts and an additional interactive Jupyter notebook. The scripts:  

**1) Select a random subset of the occupied voxels, identify statistically significant co-occupancy relationships between these voxels, and cluster them to produce initial blocks** 
 
```
siren sketch_communities --help
usage: siren sketch_communities [-h] --voldir VOLDIR --threads THREADS
                                  [--bin BIN] [--outdir OUTDIR] [--posp POSP]
                                  [--posp_factor POSP_FACTOR] [--negp NEGP]
                                  [--negp_factor NEGP_FACTOR]

optional arguments:
  -h, --help            show this help message and exit
  --voldir VOLDIR       Directory where (downsampled) volumes are stored
  --threads THREADS     Number of threads for multiprocessing
  --bin BIN             Threshold at which to binarize maps
  --outdir OUTDIR       Directory where outputs will be stored
  --posp POSP           P value threshold before Bonferroni correction for
                        positive co-occupancy
  --posp_factor POSP_FACTOR
                        Factor by which to multiply Bonferroni-corrected
                        p-value for positive co-occupancy
  --negp NEGP           P value threshold before Bonferroni correction for
                        negative co-occupancy
  --negp_factor NEGP_FACTOR
                        Factor by which to multiply Bonferroni-corrected
                        p-value for negative co-occupancy

```  
e.g.
  
```
siren sketch_communities --voldir 00_aligned/epoch_29/reconstruct_000000/downsampled --threads 20 --bin 1 --outdir 01_siren 
```  
The ```--posp```, ```---posp_factor```, ```--negp```, and ```--negp_factor``` flags are tunable parameters; while we have set default values that typically work well, users may wish to screen a range of these values to find the optimal set of values for their data. In particular, loosening some of these parameters might be beneficial in cases of extensive conformational heterogeneity.


**2) Query every voxel against each initial block to produce expanded blocks** 
  
```
siren expand_communities --help
usage: siren expand_communities [-h] --config CONFIG --blockdir BLOCKDIR
                                  --threads THREADS [--exp_frac EXP_FRAC]
                                  [--posp POSP] [--posp_factor POSP_FACTOR]
                                  [--negp NEGP] [--negp_factor NEGP_FACTOR]

optional arguments:
  -h, --help            show this help message and exit
  --config CONFIG       Path to sketch_communities.py config file
  --blockdir BLOCKDIR   Path to directory where segmented blocks are stored
  --threads THREADS     Number of threads for multiprocessing
  --exp_frac EXP_FRAC
  --posp POSP           P value threshold before Bonferroni correction for
                        positive co-occupancy
  --posp_factor POSP_FACTOR
                        Factor by which to multiply Bonferroni-corrected
                        p-value for positive co-occupancy
  --negp NEGP           P value threshold before Bonferroni correction for
                        negative co-occupancy
  --negp_factor NEGP_FACTOR
                        Factor by which to multiply Bonferroni-corrected
                        p-value for negative co-occupancy
```  
 e.g.   
   
 ```
siren expand_communties --config 00_sketch/config.pkl --blockdir 00_sketch --threads 20
 ```  

As before, the ```--posp```, ```--posp_factor```, ```--negp```, and ```--negp_factor``` parameters are tunable, as is the ```exp_frac``` value. 

## Analysis of results
Users are recommended to first view the structural blocks directly in ChimeraX, and re-run if necessary with updated parameters. When a suitable set of parameters has been identified, the interactive Jupyter notebook generated by ```expand_communities``` can be used to quantify and plot block occupancies, and identify subsets of the original volume ensemble with high occupancy of specific blocks of interest. 
