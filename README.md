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
4. Use `testsuite_list` to find testsuite for your test
5. Run package's testsuite: `testsuite_run test_(name_of_package)_run`
6. Wait and take a coffee
7. Take another one :)
8. Submit results: `testsuite_submit test_(name_of_package)_run` (we are in AFTER state!
9. When submission has terminated take note of the output:

```
  AFTER
  info: Submiting results of test_php53-run from s390vsw137.suse.de
  info: Submiting results of test_php53-run from nero.qam.suse.de
  info: Submiting results of test_php53-run from ceasar.qam.suse.de
  info: submission for s390vsw137.suse.de (suse_sles-modules-11-SP4-s390x): http://qadb2.suse.de/qadb/submission.php?submission_id=1427455
  info: submission for nero.qam.suse.de (suse_sles-modules-11-SP4-i586): http://qadb2.suse.de/qadb/submission.php?submission_id=1427459
  info: submission for ceasar.qam.suse.de (suse_sles-modules-11-SP4-x86_64): http://qadb2.suse.de/qadb/submission.php?submission_id=1427467
```

10. `downgrande` (I'm going into BEFORE state)
11. Re-run testsuite: `testsuite_run test_(name_of_package)_run`
12. Submit BEFORE results `testsuite_submit test_(name_of_package)_run`
13: Save the outputs for the BEFORE results as well

```
  BEFORE
  info: Submiting results of test_php53-run from s390vsw137.suse.de
  info: Submiting results of test_php53-run from nero.qam.suse.de
  info: Submiting results of test_php53-run from ceasar.qam.suse.de
  info: submission for s390vsw137.suse.de (suse_sles-modules-11-SP4-s390x): http://qadb2.suse.de/qadb/submission.php?submission_id=1427797
  info: submission for nero.qam.suse.de (suse_sles-modules-11-SP4-i586): http://qadb2.suse.de/qadb/submission.php?submission_id=1427801
  info: submission for ceasar.qam.suse.de (suse_sles-modules-11-SP4-x86_64): http://qadb2.suse.de/qadb/submission.php?submission_id=1427805

```

# Source code change review

3 things to investigate

1. `NEW_VERSION_OR_NEW_PACKAGE`
2. `ALL_TRACKED_ISSUES_DOCUMENTED`
3. `HAS_UNTRACKED_CHANGES`

## NEW_VERSION_OR_NEW_PACKAGE

Check something like this in sources.diff

```
old:
----
  autoyast2-4.0.55.tar.bz2

new:
----
  autoyast2-4.0.61.tar.bz2
```

or use `rpm -qa <your_pack>` before and after update

Exmple of a new version:

```
BEFORE: nodejs6-6.12.2-11.8.1
AFTER:  nodejs6-6.14.1-11.12.1
```
Example of not a new version:

```
BEFORE: python3-base-3.4.6-24.1
AFTER:  python3-base-3.4.6-25.7.1
```

In practice: if the numbers before the dash change I have a new version.

If I have a new version, in my report i can set:

`NEW_VERSION_OR_NEW_PACKAGE: YES`

otherwise

`NEW_VERSION_OR_NEW_PACKAGE: NO`

## ALL_TRACKED_ISSUES_DOCUMENTED

The expectation is to find a reference for those at the source.diff file.
If all my bugs have a reference in source.diff, in my report i can set:

 `ALL_TRACKED_ISSUES_DOCUMENTED: YES`
 
 otherwise:
 
 `ALL_TRACKED_ISSUES_DOCUMENTED: NO`
 
 
## HAS_UNTRACKED_CHANGES


# sources.diff

A lot of info to understand sources.diff

* the **original** file is preceded by "---" 
* the **new** file is preceded by "+++". 


```
changes files:
--------------
--- icewm.changes	<--- original file 
+++ icewm.changes	<--- new file
```
__________________________________________________

Following this are one or more change **hunks** that contain the line differences in the file. 
* The **unchanged**, contextual lines are preceded by **a space character**.
* **addition** lines are preceded by a **plus sign**.
* **deletion** lines are preceded by a **minus sign**.

below an example of chunk:

```
@@ -1,0 +2,6 @@	<--- chunk range information
+Fri Sep  7 06:51:23 UTC 2018 - qzheng@suse.com
+
+- Delete icewm.desktop and rename icewm-session.desktop to
+  icewm.desktop to fix the upgrade issue ( bsc#1096917 ).
+
+-------------------------------------------------------------------
```

__________________________________________________

A hunk begins with **range information** and is immediately followed with the **line additions**, **line deletions**, and any number of the contextual lines. 

The **range information** is surrounded by **double-at signs**, and combines onto a single line what appears on two lines in the context format

```
@@ -1,0 +2,6 @@
```

The format of the range information line is as follows:

```
@@ -l,s +l,s @@ optional section heading
```

* The range for the hunk of the **original** file is preceded by a **minus symbol**
* The range for the **new** file is preceded by a **plus symbol**. 
* Each hunk range is of the format l,s:
	* l is the starting line number 
	* s is the number of lines the change hunk applies to for each respective file. 
	
* In many versions of GNU diff, each range can omit the comma and trailing value s, in which case s defaults to 1. Note that the only really interesting value is the l line number of the first range; all the other values can be computed from the diff.

____________________________________________________

Below a complete example of a sources.diff file 

there are 3 changed files

1. icewm.changes
2. icewm.spec
3. _patchinfo (a new file)

```
changes files:
--------------
--- icewm.changes
+++ icewm.changes
@@ -1,0 +2,6 @@
+Fri Sep  7 06:51:23 UTC 2018 - qzheng@suse.com
+
+- Delete icewm.desktop and rename icewm-session.desktop to
+  icewm.desktop to fix the upgrade issue ( bsc#1096917 ).
+
+-------------------------------------------------------------------

spec files:
-----------
--- icewm.spec
+++ icewm.spec
@@ -225,6 +225,9 @@
 
 %suse_update_desktop_file %{buildroot}%{_datadir}/xsessions/icewm-session.desktop
 
+mv -f %{buildroot}%{_datadir}/xsessions/icewm-session.desktop %{buildroot}%{_datadir}/xsessions/icewm.desktop
+ln -s icewm.desktop %{buildroot}%{_datadir}/xsessions/icewm-session.desktop
+
 touch %{buildroot}%{_sysconfdir}/alternatives/default-xsession.desktop
 ln -s %{_sysconfdir}/alternatives/default-xsession.desktop %{buildroot}%{_datadir}/xsessions/default.desktop
 

other changes:
--------------

new:
----
  _patchinfo

other changes:
--------------

++++++ _patchinfo (new)
--- _patchinfo
+++ _patchinfo
@@ -0,0 +1,11 @@
+<patchinfo incident="8643">
+  <issue tracker="bnc" id="1096917">[RPi3][Build7.1] polkit-gnome-authentication-agent-1 not starting in xdm/icewm - firewall-config Authorization failed, unable to reboot/shutdown</issue>
+  <category>recommended</category>
+  <rating>important</rating>
+  <packager>zhengqiang</packager>
+  <description>This update for icewm fixes the following issues:
+
+- Renamed icewm-session.desktop to icewm.desktop to fix a upgrade issue (bsc#1096917).
+</description>
+  <summary>Recommended update for icewm</summary>
+</patchinfo>

new:
----
  _patchinfo

other changes:
--------------

++++++ _patchinfo (new)
--- _patchinfo
+++ _patchinfo
@@ -0,0 +1,11 @@
+<patchinfo incident="8643">
+  <issue tracker="bnc" id="1096917">[RPi3][Build7.1] polkit-gnome-authentication-agent-1 not starting in xdm/icewm - firewall-config Authorization failed, unable to reboot/shutdown</issue>
+  <category>recommended</category>
+  <rating>important</rating>
+  <packager>zhengqiang</packager>
+  <description>This update for icewm fixes the following issues:
+
+- Renamed icewm-session.desktop to icewm.desktop to fix a upgrade issue (bsc#1096917).
+</description>
+  <summary>Recommended update for icewm</summary>
+</patchinfo>

new:
----
  _patchinfo

other changes:
--------------

++++++ _patchinfo (new)
--- _patchinfo
+++ _patchinfo
@@ -0,0 +1,11 @@
+<patchinfo incident="8643">
+  <issue tracker="bnc" id="1096917">[RPi3][Build7.1] polkit-gnome-authentication-agent-1 not starting in xdm/icewm - firewall-config Authorization failed, unable to reboot/shutdown</issue>
+  <category>recommended</category>
+  <rating>important</rating>
+  <packager>zhengqiang</packager>
+  <description>This update for icewm fixes the following issues:
+
+- Renamed icewm-session.desktop to icewm.desktop to fix a upgrade issue (bsc#1096917).
+</description>
+  <summary>Recommended update for icewm</summary>
+</patchinfo>

new:
----
  _patchinfo

other changes:
--------------

++++++ _patchinfo (new)
--- _patchinfo
+++ _patchinfo
@@ -0,0 +1,11 @@
+<patchinfo incident="8643">
+  <issue tracker="bnc" id="1096917">[RPi3][Build7.1] polkit-gnome-authentication-agent-1 not starting in xdm/icewm - firewall-config Authorization failed, unable to reboot/shutdown</issue>
+  <category>recommended</category>
+  <rating>important</rating>
+  <packager>zhengqiang</packager>
+  <description>This update for icewm fixes the following issues:
+
+- Renamed icewm-session.desktop to icewm.desktop to fix a upgrade issue (bsc#1096917).
+</description>
+  <summary>Recommended update for icewm</summary>
+</patchinfo>

new:
----
  _patchinfo

other changes:
--------------

++++++ _patchinfo (new)
--- _patchinfo
+++ _patchinfo
@@ -0,0 +1,11 @@
+<patchinfo incident="8643">
+  <issue tracker="bnc" id="1096917">[RPi3][Build7.1] polkit-gnome-authentication-agent-1 not starting in xdm/icewm - firewall-config Authorization failed, unable to reboot/shutdown</issue>
+  <category>recommended</category>
+  <rating>important</rating>
+  <packager>zhengqiang</packager>
+  <description>This update for icewm fixes the following issues:
+
+- Renamed icewm-session.desktop to icewm.desktop to fix a upgrade issue (bsc#1096917).
+</description>
+  <summary>Recommended update for icewm</summary>
+</patchinfo>

```
# Documentation

1. Set the summary to **PASSED** or **FAILED**.
2. If the Summary is **FAILED**, document why it failed on the **comment: (none)** field.
3. Rearrange and add new bugs (if any ) on the **BUGS SUMMARY** section.
4. Go to the **Suggested Test Plan Reviewer** and delete the word Suggested.
5. **Remove every Example** on all sections of the report.
6. Write a summary of the installation tests on the **INSTALL TESTS SUMMARY** section. 
```
INSTALL TESTS SUMMARY

PASSED on all architectures
```
7. Write a summary of the regression tests on the **REGRESSION TEST SUMMARY** section.
```
REGRESSION TEST SUMMARY:

Run python3 testsuite on all hosts. No regression.
```
8. save: ':wq'
9. `commit`
10. Keep the link of the report and send this link to the corresponding **Test Plan Reviewer**
11. Go on IRC and find the corresponding Test Plan Reviewer. Ask him to **final check** your report before submitting it
12. In case there was **no Suggested Test Plan Reviewer provided**, check the symbol next to the ID on SUSE Maintenance. If the symbol is a bug you have to ask on the #maintenance IRC channel for someone to review it,else,if it is a shield, ask on the #security IRC for a review.If you see the indication noreview you can skip the Test Plan Review12. 

# [2018-09-21] Refhost (SLEs15)

# [2018-09-21] Localization 

# [2018-09-11] X crashed

1. https://it.opensuse.org/SDB:Configurazione_schede_video

2. https://forums.opensuse.org/showthread.php/438705-opensuse-graphic-card-practical-theory-guide-users

```
➜  ~ cat /etc/SUSE-brand 
openSUSE
VERSION = 15.0
➜  ~ /sbin/lspci -nnk | grep VGA -A2
00:02.0 VGA compatible controller [0300]: Intel Corporation Device [8086:5926] (rev 06)
	Subsystem: Dell Device [1028:075b]
	Kernel driver in use: i915
➜  ~ 
```

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
   
