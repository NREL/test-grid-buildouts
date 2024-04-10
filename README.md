Jupyter notebooks for synthetic grid buildouts
=================================================================

<img src="/docs/images/cfs_after_cutoff_and_exclusions.png" width="400" height="400"> <img src="/docs/images/buildout_sites.png" width="400" height="400">

### description
* set of notebooks for creating buildouts of synthetic transmission grids with various levels of renewable energy. 
can be used to augment any test grid from Texas A&M electric grid repository https://electricgrids.engr.tamu.edu/electric-grid-test-cases/
with varying levels of wind/solar generation. generated renewable energy timeseries (actuals and scenarios) can be used for various 
transmission grid operation problems. highly customizible workflow is split into two major parts:

###### tamu200_buildout_part1.ipynb
###### tamu200_buildout_part1_kestrel.ipynb (for use on Kestrel)
1. reading in ACTIVSg grid files
2. setting up SAM & reV
3. running reV
4. applying exclusion masks
5. saving data for part2 


###### tamu200_buildout_part2.ipynb
###### tamu200_buildout_part2_kestrel.ipynb (for use on Kestrel)
1. parse ACTIVSg grid files
2. filter sites by capasity factors
3. filter by inclusion fraction
4. determine buildable/connectable buses
5. select sites and aggregate timeseries
6. modify and save new grid files 

* additional notebooks:

###### buildout_scenarios_v2.ipynb
1. load grid with selected buildout
2. generates scenariosi, e.g., for stochastic optimization simulations

###### viz_buildout.ipynb
1. interactive vizualization of grid buildouts
2. work in progress


### installing of grid-buildouts workflow

**_NOTE:_** since grid-buildouts workflow require access to WTK and NSRDB datasets on Eagle or Kestrel HPC.


**_NOTE:_** we'll use /projects/&lt;allocation&gt;/&lt;username&gt; as root dir in example commands below.
please, change paths accordingly

**_NOTE:_** use **env.yml** on Eagle (nrel-rev=0.7.0); use **env2.yml** on Kestrel (nrel-rev=0.8.2)

```
ssh eagle
cd /projects/<allocation>/<username>
git clone https://github.nrel.gov/isatkaus/grid-buildouts
cd grid-buildouts
```

* on Kestrel
```
ssh kestrel 
cd /projects/<allocation>/<username>
git clone git@github.nrel.gov:isatkaus/grid-buildouts.git
cd grid-buildouts
```


* install conda env from env.yml (on Kestrel use env2.yml)
since home disk space is limited to 50G on Eagle, use prefix to install environment to specified location

* (optional) also, to save home dir space even futher, we'll use custom dir for conda's cashe.
by default this location is set by the conda module file to ~/.conda-pkgs

```
export CONDA_PKGS_DIRS=/scratch/<username>/.conda-pkgs
module load conda
cd /projects/<allocation>/<username>/conda-envs/
mamba env create --prefix buildouts --file /projects/<allocation>/<username>/grid-buildouts/env.yml
```

* on Kestrel

```
export CONDA_PKGS_DIRS=/scratch/<username>/.conda-pkgs
module load anaconda3
module load mamba
cd /projects/<allocation>/<username>/conda-envs/
mamba env create --prefix buildouts --file /projects/<allocation>/<username>/grid-buildouts/env2.yml
```


**_NOTE:_** `--prefix` install will create two lines in `conda env list` output 
(this is normal, both point to the same place ), e.g.,

```
/projects/<allocation>/<username>/conda-envs/buildouts
/lustre/eaglefs/projects/<allocation>/<username>/conda-envs/buildouts
```



* once env installed, next two steps are optional:
1. to make conda look for envs in custom dir, do this once. it will modify ~/.condarc file and you'll be 
able to activate conda env using only short directory name (buildouts) and not it's entire path  
```
conda config --append envs_dirs /projects/<allocation>/<username>/conda-envs/ 
```
2. to make command prompt to not contain full env prefix (/projects/&lt;allocation&gt;/&lt;username&gt;/conda-envs/buildouts) but just 
the short env name (buildouts), modify conda's config further ({name} below is literal):  
```
conda config --set env_prompt '({name})'
```

* now can activate it without full path prefix, and command promt will show only env's name, e.g., (buildouts)[isatkaus@el2 conda-envs]$ 
```
conda activate buildouts 
```


* we also need to add a couple more packages to our env: pywtk and powerscenarios

```
git clone https://github.com/NREL/pywtk.git
git clone -b dev https://github.com/NREL/powerscenarios.git
```

**_NOTE:_** pytwk is a bit old and has python2 print statements in pywtk/pywtk/fill_tz.py file
and when using python3, we need to overwrite those

```
cp powerscenarios/patches/fill_tz.py pywtk/pywtk/fill_tz.py
cd pywtk
python setup.py install
cd ../powerscenarios
pip install -e .
```
* add "buildouts" env to Jupyter's kernel select drop down
```
python -m ipykernel install --user --name buildouts --display-name "buildouts"
```


### to use notebooks on Europa: https://europa.hpc.nrel.gov/
* add "buildouts" env to Jupyter's kernel select drop down
```
python -m ipykernel install --user --name buildouts --display-name "buildouts"
```
* in your browser go to https://europa.hpc.nrel.gov/ 
open tamu200_buildout_part1.ipynb notebook and select "buildouts" kernel


### to use notebooks on compute node (Eagle)
```
ssh eagle
salloc --time=12:00:00 --account=<allocation> --nodes=1
```

* note the &lt;nodename&gt;, e.g., [isatkaus@r1i2n27 ~] -> nodename = r1i2n27, we'll use it when 
accessing the notebook

```
module load conda
conda activate buildouts
jupyter-notebook --no-browser --ip=$(hostname -s)
```

* note the url, e.g.,http://127.0.0.1:8888/?token=21eb359255fc9c793c098f03f9803e8c731abae3124db3d8 
and in a new terminal on your local mac/pc (possibly use a different port if 8888 is taken by other app)
do port forward to access running Jupyter (use the nodename from previous step, e.g., r1i2n27)
```
ssh -N -L 8888:<nodename>:8888 eagle 
```

* in your browser go to (use your token obtained above)
http://127.0.0.1:8888/?token=21eb359255fc9c793c098f03f9803e8c731abae3124db3d8

### to use notebooks on compute node (Kestrel)
```
ssh kestrel 
salloc --time=12:00:00 --account=<allocation> --nodes=1
```
* or use partial node:
```
salloc -p shared -n 10 -t 1-00 --account=<allocation> --mem=20GB
```

* note the &lt;nodename&gt;, e.g., [isatkaus@x1008c0s1b0n0 ~]$ -> nodename = x1008c0s1b0n0, we'll use it when 
accessing the notebook

```
module load anaconda3 
conda activate buildouts
jupyter-notebook --no-browser --ip=$(hostname -s)
```

* note the url, e.g.,http://127.0.0.1:8888/?token=21eb359255fc9c793c098f03f9803e8c731abae3124db3d8 
and in a new terminal on your local mac/pc (possibly use a different port if 8888 is taken by other app)
do port forward to access running Jupyter (use the nodename from previous step, e.g., x1008c0s1b0n0)
```
ssh -N -L 8888:<nodename>:8888 kestrel 
```

* in your browser go to (use your token obtained above)
http://127.0.0.1:8888/?token=21eb359255fc9c793c098f03f9803e8c731abae3124db3d8


### Data
* sythetic grid defining files can be downloaded from
https://electricgrids.engr.tamu.edu/electric-grid-test-cases/

* for other supplemental files, e.g., exclusion mask, might need to obtain read access to hpcapps allocation

* for question regagring data, or any other fuctionality of the notebooks
please email me at ignas.satkauskas@nrel.gov or text me on Teams

