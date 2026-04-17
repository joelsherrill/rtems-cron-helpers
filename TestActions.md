# Cron Sweeper Flow

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
  if autoconf
    bootstrap rtems

if rsb or rtems updated or forced
  build  and test (if possible) the following BSPs with Debug on/off
    and SMP on/off if supported
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


