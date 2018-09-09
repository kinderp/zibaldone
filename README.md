# zibaldone
my dayly notes

## Looking for a new update for me

Go to https://maintenance.suse.de/overview/  

`ibs qam list`

## Assign to myself an update

`ibs qam assign SUSE:Maintenace:ID:RR`

## Check if the update is assigned to myself

`ibs qam my`

## Let's start to work on my update

`mtui -r SUSE:Maintenance:ID:RR`

## First checkings

0. Check the hosts I'm using for testing and their environments (products and architectures) and take note.
   * `list_hosts`
   * `list_products`
   
   It is useful to know because some bugs in my update could not be reproduceble in some architectures. I can know that just only reading all comments in bug's bugzilla page (see point 2) 

1. Check SMELT CHECKER field in update's template, it is important to know that because it could be a good reason to reject before testing. So if there is a smelt checker, I should comunicate as soon as possible to the `maintenance coordinator` so that he approves that they are not something to worry and I can start my update.

2. Check the bug lists for the update.
   * Read all comments in the bugzilla page because maybe PFT for some bugs has been provided, so it is not necessary to reproduce.
   * Go to qam.suse.de and search for the package you are testing or the number of the bug, there I can see old reports and what other people did that could help me with the bug.
   

At this time I should have a good idea on how to reproduce bugs or know the problems because of they can't be reproduced
   
In short for a specific bug I will be in these following situations:

* Reproducer present and works

* Reproducer Present but does not work

* Reproducer not Present

* PTF provided


# Installation tests

Installation tests consist of 4 command steps.

1. `prepare`
2. `update`
3. `export`
4. `edit`

In more detail:

* `prepare`: It will install all the packages needed to perform my testing update in all the machines that mtui has connected to.
    * After `prepare` has terminated it is import to run `list_packages`.
    * `list_packages` shows the status of the packages for the actual testing update. The packages, in all the machines, must be in a `update_needed` or `not_installed` status.
    * Take notes which packs are in `update_needed` and which one in `not_installed` status.
* `update`: It will apply the update.
    * After `update` has terminated the packages that were in `update_needed` should pass in `updated`
    * After `update` has terminated the packages that were in `not_installed` should remain in `not_installed`
    * It the two previous condition are not verified, I can usee `terms gnome` to open terminals in the machines and fix the issue.
* `export`: It will save all the installation results automatically to my report 
* `edit`: Check the results are PASSED and all it's good.

# Reproducing bugs

After `first checking step` I should know quite a bit about my bugs.

As stated in gold rules doc:

> try to reproduce every bug and document this in the template. 
> Documentation includes:
>
> 1. setup for test
> 2. test actions and results before updating

as previous suggested, bugs can be in these 4 status:

* Reproducer present and works

* Reproducer Present but does not work

* Reproducer not Present

* PTF provided


All bugs must be reproduced BEFORE and AFTER the `update`.

If the test is in a AFTER (update) state, I can pass to BEFORE (update) state using: `downgrade -t host.bla.bla`

In short:

0. Check how to document these different situations using Orestis' guide (paragraph Reproducing a bug)
1. Reproduce a bug in just a single machine (I'm hypothesizing to be in AFTER state).
2. `edit` my report as suggested at point 0 usign the AFTER clause
3. `downgrade -t the.machine.i_am_reproducing_bug` 
4. Re-reproduce the bug in BEOFRE state
5. `edit`the report using the BEFORE clause
6. `commit` my reproducing bugs results.

# Regression Testing (testsuite are available)

Testsuites must be run AFTER and BEFORE the update.
Supposing test's state is AFTER now.

0. Check testsuite is present for the package in testing: `run zypper se qa_test_(name_of_package)`
1. Install testsuite: `run zypper -n in qa_test_(name_of_package)`
2. Check the installation: `run zypper se qa_test_(name_of_package)`
3. If you don't remember the state of your test use `list_packages`
4. 
# [2018-09-04] *first day training*

1. load the test report from svn with the command

   `mtui -r SUSE:Maintenance:7720:171354`

```
  this command also connects to a bunch of machines
  wiht a corresponding architectures useful to test
  the updates
  
  finally it gives us a mtui shell prompt for an interactive session.
```
  

2. we can check the hosts that we are connected

   `list_hosts`

```
   so we can remove some hosts and disconneting from those with
   the command
   
   remove_hosts -t <hostname>
   
   e.g. is better to remove hosts like teradata.bla.bla
```

3.  So now we can check the status of the packages involved with the updates


    `list_packages`

```
    this show us the status of the packages in all machines

    It is import to check if there is some machine where the packages 
    status is NOT_INSTALLED because in this case we must prepare 
    our updates.

    The status of the packages in this step can also be UPDATE_NEEDED
    but is a good thing.
```

4. So if there are in some machines packages in a NOT_INSTALLED status
   we can solve with the command

   `prepare`

5. (a) After the prepare command we can re-check the status of the packages 

   `list_packages`

```
   Now we must not have any package in a NOT_INSTALLED status.
   We can see zypper's logs in install_logs dir and
   use terms gnome to open terminals into the VMs and fix the problem.

   e.g. We have had a conflit between php5 and php7 so we have removed
        php7 and crossed the fingers.
```

5. (b) If something has gone wrong at point 5(a) and we have fixed that, we can
    re-exeute 
    
    `prepare`
   
   
6. If prepare has worked (all the packages in all machines are in a
   UPDATE_NEEDED status) we can apply the updates with the command
   
   `update`
   
   
7. (a) We can check update results with the usual command

   `list_packages`
   
   ```All packages in all machines must be now in a UPDATED status```
   
7. (b) If something has gone wrong at point 7 (a) ???

8.  So now we have finished the INSTALLESTION_STEPS and we can export
     our report.
    
```
    What Does report do (from mtui docs):
    Exports the gathered update data to template file. This includes the pre/post package versions and the update log. An output file can be specified; if none is specified, the output is written to the current testing template. Refhost zypper installation logs are exported to subdir per refhost.
```
    
9. So now we can edit our report with some comments.

```
   At this point Is a good idea to check is all the tests in the 
   "Test results by product-arch" field are PASSED, maybe we have
   escaped some error at point 7.
```
   
