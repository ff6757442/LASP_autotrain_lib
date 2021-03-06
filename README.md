# LASP SSW-DFT-NN auto-train python-lib

## Author
Original: ZPLiu's Group (SDHuang, ZPLiu et al)

Modified: James.Misaka.Bourbon.Liu

Last Change: 2022-07-14

Version: V1.1

## Program Structure

![LASP_PyLib_Architecture](image/auto_workflow.png)

![auto_dependance](image/auto_dependance.png)

![auto_workflow](image/auto_workflow.png)


## How to run it directly

### 1. modify console file

#### 1.1 some following are parameters often need to change

```shell
StartfromVASP 0   # 0 start from SSW sampling provided with NN pot , 1 start from allstr.arc-0 in VASP dir, which is often used to train the first NN pot
Nbad   40    # structures for VASP every cycle
cpupernode 96   # CPU total core (not suggested running between 2 or more CPUs)
SSWcheckcycle  600   # SSW time clock 600 seconds

%block cpuperjob
SSW  24         # cores per SSW job
VASP 24         # cores per VASP job
NN   0          # should be designated in jobs.sh
%endblock cpuperjob
```

#### 1.2 provide the binary program

```sh
%block prog
SSW  /home10/bin/lasp-1.0-release/lasp
VASP  /home10/bin/lasp-1.0-release/lasp
VASPgamma  /home10/bin/lasp-1.0-release/lasp.gamma
NN  /home10/bin/lasp-1.0-release/lasp
%endblock prog
```

#### 1.3 provide the element name in console file

```sh
%block base
O   0.0
H   0.0
%endblock base
```

### 2. modify jobs.sh

1. make sure the name of NN pot is correct, e.g. sed -i 's/H2O/PtOH.pot/g' jobs.sh
2. modify the number of cycles, default is 100, in: for i in {1..100}
3. modify the cpu/cores required for your computing cluster (modify it in jobs.sh)

### 3. create arc file for SSW sampling:  allstr-ini.arc
   in SSW/sourcedir/allstr-ini.arc

   you may get allstr-ini.arc from the examples of structure which you need to add and train in your pot


### 4. check NN directory
   In rootdir/NN you should prepare:

```shell
   lasp.in   # lasp_NNtrain input file
   H2O.pot          # not required if start from scratch
   H2O.input        # if start from scratch, use "newrun" for pot
   TrainStr.txt     # if start from scratch, just creat an empty file
   TrainFor.txt     # if start from scratch, just creat an empty file
   adjust_factor  # can ignore
```

lasp already has a lot of Train*.txt files for different systems please first download TrainStr.txt TrainFor.txt from www.lasphub.com

### 5. make sure add python exec path in jobs.sh

You'd better have anaconda env in your server, otherwise you can use intel-python:

You can use intel-python by define it in .bashrc:

```sh
export PYTHONPATH=/data/apps/intel/intelpython3/bin:$PYTHONPATH
```

### 6. qsub jobs.sh (or sbatch jobs_local.slurm)

## Other scripts

1. related to traindata
   1. shiftformat.py: arc2train or train2arc usage
   2. vasp2lasptrain.py: vasp-dft result directly to TrainStr.txt and TrainFor.txt
   3. cut_traindata.py: cut TrainStr/TrainFor by 
   4. traindata_analysis.py: print-out statistic infomation of TrainStr/TrainFor
2. related to arc_data
   1. findGM.py: find top100(can be set) global minimum structure from SSW result
   2. splitarc_auto.py: split all-str arc file to one-str arc file (or dft-project dir)
   3. nodejob.py and nodejob_coor.py: collect all-str from SSW to VASP-DFT
   4. collect_vasp_label.py: collect and screen all-str from SSW to VASP-DFT (not test)
   5. screen_data.py: from auto.py, used to screen all-str from SSW result (not test)
3. related to coordination patterns
   1. dft_setting dir: used for coor_verlet_sample.py
   2. coor_verlet_sample.py: two-steps dynamic verlet sampling based on stucture similarity described by coordination patterns
   3. update_patterns.py: generate coordination patterns of all-str (and add to exist database)
      1. parallel version is still coding/refining, this version have lower speed

## some tips

1. SSW/sourcedir have input.i for SSW-NN input file, may need to check
2. remember check SSW VASP NN dir before finally running
3. Remember check auto.py(SSW_choosing_mode) and jobs.sh before qsub/sbatch
4. Coordination-Patterns method waiting for using
5. auto.py and SSW-DFT-NN auto still need to be tested
