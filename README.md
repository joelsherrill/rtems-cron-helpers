# rtems-cron-helpers

Introduction
============

This is a set of scripts to assist in setting up and running cron jobs
to test building RTEMS tools and BSPs.

There is also a subdirectory of simpler single-purpose scripts which
are intended to be used when making a test build of all tools before
turning the job over to cron.

Example
=======
The rtems-cron-prepdir script is used to setup a working directory
for rtems-cron-runner.  This is an example:

~~~~
./rtems-cron-prepdir -g -V 6 -D ${HOME}/rtems-cron-6

./rtems-cron-helpers/rtems-cron-runner -f -V 6 -D  ${HOME}/rtems-cron-6 
~~~~

And one for setting up to test "next" tools

~~~~
./rtems-cron-prepdir -g -V 7 -D ${HOME}/rtems-cron-next

./rtems-cron-helpers/rtems-cron-runner -f -V 7 -s next -D ${HOME}/rtems-cron-next
~~~~



Before executing the time consuming rtems-cron-runner script, it is
important to make sure that the host computer is properly.

If anything fails in a way that does not result in mail being sent
to build@rtems.org, then you will have to look in the long files.

The rtems-cron-runner script takes long enough enough to run (> 24
hours for a full sweep) that you should "nohup" it.

crontab Entries
===============

~~~~
  # Run update-cron-helpers at 11:50PM every day
  50 23 * * * /usr/local/bin/update-cron-helpers

  # RTEMS Tools Coverity every 3 hours at nn:00
  0 0,3,6,9,12,15,18,21 * * * /usr/bin/flock -n /tmp/tools.lock ~/rtems-cron-helpers/rtems-tools-cron-run-coverity -v >${HOME}/nohup-rtems-tools.out 2>&1 </dev/null

  # Newlib Coverity every 3 hours at nn:15
  15 0,3,6,9,12,15,18,21 * * * /usr/bin/flock -n /tmp/newlib.lock ~/rtems-cron-helpers/rtems-newlib-run-coverity >${HOME}/nohup-rtems-newlib.out 2>&1 </dev/null

  # RTEMS Coverity every 3 hours at nn:00
  0  1,4,7,10,13,16,19,22 * * * /usr/bin/flock -n /tmp/rtems.lock ~/rtems-cron-helpers/rtems-cron-run-coverity >${HOME}/nohup-rtems.out 2>&1 </dev/null
~~~~

Release Candidate Testing
=========================

~~~~
  rm -rf rtems-cron-5.1-rc2 
  ~/rtems-cron-helpers/rtems-cron-prepdir \
    -D ${HOME}/rtems-cron-5.1-rc2 -r \
    -R https://ftp.rtems.org/pub/rtems/releases/5/rc/5.1-rc2/ -t 5.1-rc2
  nohup time sh -x ~/rtems-cron-helpers/rtems-cron-runner \
    -v -g -f -V 6 -D  /home/joel/rtems-cron-5.1-rc2 \
    >nohup-510rc2-2.out </dev/null 2>&1 &
~~~~

~~~~
  rm -rf rtems-cron-6.1-rc7
   ~/rtems-cron-helpers/rtems-cron-prepdir -D /home/joel/rtems-cron-6.1-rc8 -r -R https://ftp.rtems.org/pub/rtems/releases/6/rc/6.1-rc8/ -t 6.1-rc8

   nohup time sh -x ~/rtems-cron-helpers/rtems-cron-runner -v -g -f -V 6 -D  /home/joel/rtems-cron-6.1-rc8 >nohup-6.1rc8.out </dev/null 2>&1 &

~~~~

~~~~
  rm -rf rtems-cron-6.1-rc8
   ~/rtems-cron-helpers/rtems-cron-prepdir -D /home/joel/rtems-cron-6.1 -r -R https://ftp.rtems.org/pub/rtems/releases/6/6.1/ -t 6.1

   nohup time sh -x ~/rtems-cron-helpers/rtems-cron-runner -v -g -f -V 6 -D  /home/joel/rtems-cron-6.1 >nohup-6.1.out </dev/null 2>&1 &

~~~~


Preparation
===========

Ensure the host computer is initialized per the instructions in
the Users Guide.  This includes using sb-check.

On Cygwin, you will need to setup ssmtp before performing these
steps or results will not be emailed to build@rtems.org. Also install
cron.

On Mingw, XXX

After running "rtems-cron-prepdir", do a at least a manual build
of a single tool chain or BSP as normal. This is a good sanity 
check.  Ensure this reports results to build@rtems.org as this
seems to be a common failure. Remember you have to be subscribed to
send to an RTEMS.org mailing list.


rtems-cron-runner Actions
=========================

The script performs the following actions:

~~~~
  if "forced" selected
    all need updating and rebuilding
  else
    rebuild and test based on what changed
    Determine what has updates
      * rtems-source-builder
      * rtems-tools
      * rtems

  if RSB updated or forced
    build all tools (rtems-all)
    build dtc
    build spike
    build qemu4

  if rtems updated or forced
    bootstrap rtems

  if rsb or rtems updated or forced
    build  and test (if possible) the following BSPs
      sparc: erc32-sis leon2-sis leon3-sis
      powerpc: test_single_bsp powerpc psim
      mips: jmr3904
      riscv (SIS): griscv-sis
      riscv (Spike): rv32iac_spike rv32imac_spike rv32imafc_spike
	    rv32imafdc_spike rv32imafd_spike rv32im_spike rv32i_spike
	    rv64imac_medany_spike rv64imac_spike rv64imafdc_medany_spike
	    rv64imafdc_spike rv64imafd_medany rv64imafd_medany_spike
	    rv64imafd_spike

    build the following BSP bsets (should be all bsp bsets)
      atsamv beagleboneblack erc32 gr712rc gr740 imx7 pc
      qoriq_e500 qoriq_e6500_32 qoriq_e6500_64 raspberrypi2
      xilinx_zynq_zc702 xilinx_zynq_zc706 xilinx_zynq_zedboard

    run the rtems-bsp-builder for everything (~1700)
~~~~

rtems-cron-runner Execution Times
=================================
The set of actions performed by rtems-cron-runner has grown over 
time. These times are from early April 2020.

Xeon E3-1230 v6 @ 3.50GHz (8 cores) CentOS 7        29h52m (devel)

Times have not been updated been updated recently on the following:

Q6600 CPU @ 2.4GHz w/4              Ubuntu 18.04    10h34m  (rtbf64a)
Q6600 CPU @ 2.4GHz w/4              FreeBSD 12      10h34m  (rtbf64b)
i7-5930K CPU @ 3.50GHz w/32GB       CentOS 7         5h15m  (rtbf64c)

Coverity Scan Scripts
=====================
There is a family of scripts to aid in running Coverity Scan on RTEMS,
RTEMS Tools, and Newlib. there are base scripts to do the build and
submit the results as well as a script to run via cron to automate
running the analysis when something changes. For RTEMS Tools, there
is only the cron script to check for updates, build with analysis,
and submit results. The coverity related scripts are as follows:

+ rtems-newlib-run-coverity - runs Coverity Scan on newlib

+ rtems-run-coverity - runs Coverity Scan on RTEMS

+ rtems-tools-run-coverity - runs Coverity Scan on RTEMS Tools

+ rtems-tools-cron-run-coverity - runs from cron to check for updates
  and runs Coverity Scan on RTEMS Tools if there are any changes

+ rtems-cron-run-coverity - runs from cron to check for updates for the
  RTEMS Source Builder, RTEMS, and Newlib. As required by the updates,
  Coverity Scan analysis is performed.

If the RTEMS Source Builder is updated, then both RTEMS and Newlib must
undergo analysis. Otherwise, analysis is only performed on RTEMS if it
has been updated. Similarly for Newlib.

To Do
=====

+ (rtems-con-runner) There is no check that the script is already
  running. This prevents it from being setup to check at a frequent interval
  for updates and the need to do a partial test sweep.

+ (rtems-cron-runner) As more simulators have good test results, they should be added.

+ (rtems-cron-runner) Add build of rtems-libbsd for appropriate BSPs

+ (rtems-cron-runner) Add build of rtems-legacy-net for appropriate BSPs

+ (rtems-cron-runner) There is no way to automatically tune the --jobs
  argument to optimize it for the host computer. The setting depends on
  the number of cores, the amount of RAM, and the disk setup. With most
  computers having SSDs the disk setup doesn't seem to be the bottleneck
  that it was years ago.  But it is easy to overload a computer with 4 or 8
  GB RAM and force it to swap. This is assuming the computer is dedicated
  to testing. If it isn't, then you want to keep the load lighter. The
  long-term solution is to add a command line option corresponding to
  the --jobs parameter to various RSB tools. This lets the localization
  happen at the command line. Various settings for OAR computers which
  can be dedicated to testing

      - 12 cores (i7-5930K) w/32 GB RAM         6/12
      - 8 Xeon cores w/32 GB RAM                6/12
      - 4 Q6600 cores w/4 GB RAM                ???


