# Outreachy_Contribution_8

## Automating Workflows Using the Command Line

This contribution Explores the functions involved in the running of Galaxy workflows through the command line

The Objective of this task is to:
- Learn to use the planemo run subcommand to run workflows from the command line
- Learn to write codes for running multiple workflows concurrently or sequentially.


### Downloading the workflow from Github
**git clone https://github.com/usegalaxy-eu/workflow-automation-tutorial.git
cd workflow-automation-tutorial

This repository contains two subdirectories.
**example** - contains the tutorial.ga file 
This file defines galaxy workflow steps used in this tutorial.

**Pangolin** - contains workflow for the second exercise

### Installing and Updating Planemo
**$ python3 -m venv planemo
**$ . planemo/bin/activate
**$ pip install planemo

**$ . planemo/bin/activate
**$ pip install -U planemo

Planemo runs workflows from the command line

## Creating a job file 
The job file idefines all the inputs and parametres we want in the workflow invocation.
planemo workflow_job_init is used to create a template jobfile which we will edit.

**planemo workflow_job_init tutorial.ga -o tutorial-init-job.yml.

We run the commands below to create our datasets

  **printf "hello\nworld" > dataset1.txt
  **printf "hello\nuniverse" > dataset2.txt
  
 We then update our jobfile with the datasets we have created by editing the placeholders with 
 **vim tutorial-init-job.yml
 
 . 
 
 ## Invoking the workflow with planemo
 
 We require the following to invoke the workflow
 - galaxy url
 - galaxy API key
 - history name
 - tag (optional)
 - 
 #### code
**planemo run tutorial.ga tutorial-init-job.yml --galaxy_url <SERVER_URL> --galaxy_user_key <YOUR_API_KEY> --history_name "Test Planemo WF" --tags "planemo-tutorial"
 planemo run tutorial.ga tutorial-init-job.yml --galaxy_url <SERVER_URL> --galaxy_user_key <YOUR_API_KEY> --history_name "Test Planemo WF with no_wait" --      tags "planemo-tutorial" --no_wait

This creates a new history in the browser as shown below:
[Test Planemo run 1](https://usegalaxy.eu/u/jewelz/h/test-planemo-wf)
[Test Planemo run 2](https://usegalaxy.eu/u/jewelz/h/test-planemo-wf-1)

Using the *no_wait* flag exits the planemo run command as soon as the two datasets have been uploaded and the workflow has been scheduled

### Using Galaxy workflow and dataset IDs
