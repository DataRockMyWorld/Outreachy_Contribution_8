# Outreachy_Contribution_8

## Automating Workflows Using the Command Line

This contribution Explores the functions involved in the running of Galaxy workflows through the command line

The Objective of this task is to:

- Learn to use the planemo run subcommand to run workflows from the command line
- Learn to write codes for running multiple workflows concurrently or sequentially.


### Downloading the workflow from Github

* `git clone https://github.com/usegalaxy-eu/workflow-automation-tutorial.git.` 
* `cd workflow-automation-tutorial`

This repository contains two subdirectories.

**example** - contains the tutorial.ga file :(This file defines galaxy workflow steps used in this tutorial)

**Pangolin** - contains workflow for the second exercise  

### Installing and Updating Planemo

* `$ python3 -m venv planemo`

* `$ . planemo/bin/activate`

* `$ pip install planemo` 

* `$ . planemo/bin/activate`

* `$ pip install -U planemo`

Planemo runs workflows from the command line

### Creating a job file 
The job file defines all the inputs and parametres we want in the workflow invocation.  
planemo workflow_job_init is used to create a template jobfile which we will edit.  

* `planemo workflow_job_init tutorial.ga -o tutorial-init-job.yml`  

We run the commands below to create our datasets

  * `printf "hello\nworld" > dataset1.txt`  
  * `printf "hello\nuniverse" > dataset2.txt`  
  
 We then update our jobfile with the datasets we have created by editing the placeholders with:
 
 * `vim tutorial-init-job.yml`
  
 
 ### Invoking the workflow with planemo
 
 We require the following to invoke the workflow
 -  `galaxy url` -> https://usegalaxy.eu
 -  `galaxy API key`-> 3bd4853b3af6c5c00b292ea806597b58
 -  `history name`-> Test Planemo WF
 -  `tag (optional)`-> planemo-tutorial
 
 #### code
* `planemo run  tutorial.ga tutorial-init-job.yml --galaxy_url https://usegalaxy.eu --galaxy_user_key 3bd4853b3af6c5c00b292ea806597b58 --history_name "Test Planemo WF" --tags "planemo-tutorial"`  
 * `planemo run 2c27714425520370 tutorial-init-job.yml --galaxy_url https://usegalaxy.eu --galaxy_user_key 3bd4853b3af6c5c00b292ea806597b58 --history_name "Test Planemo WF with no_wait" --tags "planemo-tutorial" --no_wait`  

This creates a new history in the browser as shown below:

**[Test Planemo run 1](https://usegalaxy.eu/u/jewelz/h/test-planemo-wf)**   
**[Test Planemo run_no_wait](https://usegalaxy.eu/u/jewelz/h/test-planemo-wf-with-nowait)**   

Using the **no_wait** flag exits the planemo run command as soon as the two datasets have been uploaded and the workflow has been scheduled

### Using Galaxy workflow and dataset IDs

Every dataset and workflow in galaxy has a unique hexadecimal id associated with them.  
The **dataset Ids** are obtained from the **History Content API ID** under the **Dataset Information tab**, which can accessed via the **info button** in history.   
For datasets and workflows that are identical, we can reference the galaxy dataset ids in the job file and run them on the server.

Dataset 1:   
  class: File  
  `# path: dataset1.txt`  
  galaxy_id: 4838ba20a6d867654ec4d8bfbbf857c6    
Dataset 2:  
  class: File  
  `# path: dataset2.txt`  
  galaxy_id: 4838ba20a6d86765f0adb9ebe2ca3f41    
Number of lines: 3  

The workflow Id can be obtained from the **workflow URL**. 

We then insert it in the planemo run.

* `planemo run <**WORKFLOW ID**> tutorial-init-job.yml --galaxy_url <SERVER_URL> --galaxy_user_key <YOUR_API_KEY> --history_name "Test Planemo WF with Planemo" --tags "planemo-tutorial" --no_wait`


### Using Planemo Profiles
Profiles allow you to combine multiple flags together into a single profile which you can append to a command, and use all of the flags associated with that profile.

**Create planemo Profile** 

* `$ planemo profile_create planemo-tutorial --galaxy_url https://usegalaxy.eu --galaxy_user_key 3bd4853b3af6c5c00b292ea806597b58`

**Running the Workflow**

* `$ planemo run 2c27714425520370 tutorial-init-job.yml --profile planemo-tutorial --history_name "Test Planemo WF" --tags "planemo-tutorial" --no_wait`

The resulting history is : [Test_Planemo_run_with_profile](https://usegalaxy.eu/u/jewelz/h/test-planemo-wf-2)


# Automated runs of a workflow for SARS-CoV-2 lineage assignment

We first explore the pangolin folder using

`cd ../pangolin`     
`ls` to see what the folder contains     

It contains 

- data : which holds the VCF files we would run the workflow on
- vcf2lineage.ga : defines the workflow we want to run.

We then create a job file called `*vcf2lineage-job.yml` (reference previous tutorial for code)

The unique step in this automation is writing a script in the construction of the job file.   
This is to add every element of the variant calling collection.   

**Script1

import sys  
from glob import glob  
from pathlib import Path   

import yaml   

job_file_path = sys.argv[1]   
batch_directory = sys.argv[2]   

with open(job_file_path) as f:   
    job = yaml.load(f, Loader=yaml.CLoader)   

vcf_paths = glob(f'{batch_directory}/*.vcf')   
elements = [{'class': 'File', 'identifier': Path(vcf_path).stem, 'path': vcf_path} for vcf_path in vcf_paths].  
job['Variant calls']['elements'] = elements   

with open(job_file_path, 'w') as f:   
    yaml.dump(job, f)   
    
This results in the succesful invocation of the [VCF2lineage test](https://usegalaxy.eu/u/jewelz/h/vcf2lineage-test) workflow.

#### Automating vcf2lineage execution

The next step is to automate this process so we can run the workflow on each of the 10 batch*/ directories in the data/ folder.   
To execute the workflow multiple times, Just like the initial tutorial:  

We obtain the **Reference genomes dataset id and the workflow id**.  
We then modify the jobfile by adding the Reference Genome dataset id to it as **`galaxy_id`**.   

We then shell a script to   

- iterate over all the batches 
- create a job file
- invoke with planemo run
- move the processed batch to data/complete after execution.

**Script 2. 

for batch in `ls -d data/batch*`; do  

     batch_name=`basename $batch   .  
     cp vcf2lineage-job-template.yml vcf2lineage-${batch_name}-job.yml  
     python create_job_file.py vcf2lineage-${batch_name}-job.yml $batch  
     # replace with your own workflow ID below  
     planemo run f4b02af7e642e75b vcf2lineage-${batch_name}-job.yml --profile planemo-tutorial  
     sleep 300  
     nmv $batch data/complete/  
done  

We then save it as `run_vcf2lineage.sh`, make it executable by `chmod +x run_vcf2lineage.sh` and run it.

The resulting histories:


 - [VCF2lineage test](https://usegalaxy.eu/u/jewelz/h/vcf2lineage-test)
 - [CWL Target History_](https://usegalaxy.eu/u/jewelz/h/cwl-target-history)
 - [CWL Target History_1](https://usegalaxy.eu/u/jewelz/h/cwl-target-history-1)
 - [CWL Target History_2](https://usegalaxy.eu/u/jewelz/h/cwl-target-history-2)
 - [CWL Target History_3](https://usegalaxy.eu/u/jewelz/h/cwl-target-history-3)
 - [CWL Target History_4](https://usegalaxy.eu/u/jewelz/h/cwl-target-history-4)
 - [CWL Target History_5](https://usegalaxy.eu/u/jewelz/h/cwl-target-history-5)
 - [CWL Target History_6](https://usegalaxy.eu/u/jewelz/h/cwl-target-history-6)
 - [CWL Target History_7](https://usegalaxy.eu/u/jewelz/h/cwl-target-history-7)
 - [CWL Target History_8](https://usegalaxy.eu/u/jewelz/h/cwl-target-history-8)
 - [CWL Target History_9](https://usegalaxy.eu/u/jewelz/h/cwl-target-history-9)

#### Observation

Loading each dataset was very slow. The whole process took over 4 hours.    
In hindsight, i should have used the **--simultaneous_uploads** flag to upload simultaneously.     
