# zibaldone
my dayly notes

## Looking for a new update for me

Go to https://maintenance.suse.de/overview/  

`ibs qam list`

## Assign to myself an update

`ibs qam assign SUSE:Maintenace:ID:RR`

## Check if update is assigned now to myself

`ibs qam my`

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


# Installation steps



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
   
