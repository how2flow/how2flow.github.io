---
permalink: /documents/gic-t32/
title: GIC
excerpt: "Generic Interrupt Controller"
comments: false
toc: true
---

## GIC Trace32

이 문서는 Trace32 디버거 사용시, GIC 관련 유용한 내용을 정리했습니다.<br>
GICv3 기준입니다.<br>

### Peribrowser

GIC CPU interface는 ARMv8-A architecture의 시스템 레지스터 일부여서<br>
CPU attach 순간 peribrowse가능하다.<br>
그 외에, Distributor나 Redistributor 레지스터는 peri reprogram이 필요하다.<br>

peri 등록은 다음과 같다.
```
B:: sys.config.gicd type gic600
B:: sys.config.gicd base a:{gicd base address}
B:: sys.config.gicr base a:{gicr base address}
B:: per.reprogram
```
<br>

### CMM script

GICv3 하드웨어 인터럽트 동작을 확인할 수 있는<br>
CMM 스크립트

<details>
  <summary><B>Chip boot 직후(core0 only) GIC ON script</B></summary>
  <p>
  <pre><code>
/////////////////////////////
// tcc807x gicv3 init script
//
// - for using gic
//     in bare-metal
/////////////////////////////


////////////////////////
// User Configuration //
////////////////////////
&gicd_base={gicd base addr}
&gicr_base={gicd base addr}
&gicr_num_of_regs=1.


////////////////
// GIC offset //
////////////////
&sgi_offset=0x10000


/////////////
// Example //
/////////////
GOSUB GICV3_DISTIF_INIT &gicd_base
GOSUB GICV3_RDISTIF_INIT &gicr_base &gicr_num_of_regs
GOSUB GICV3_CPUIF_INIT &gicr_base
ENDDO


///////////////
// functions //
///////////////
GICV3_DISTIF_INIT:
        ENTRY &arg_gicd_ctlr
        LOCAL &rwp_true
        LOCAL &index &value

        &value=D.Long(AD:&arg_gicd_ctlr)
        &value=&value|(1<<0) ;enable G0
        &value=&value|(1<<1) ;enable G1
        &value=&value|(1<<2) ;enable G1NS
        // enable G0, G1, G1NS before enable ARE_S //
        D.S AD:&arg_gicd_ctlr %LE %Long &value

        &value=&value|(1<<4)
        &value=&value|(1<<5)
        D.S AD:&arg_gicd_ctlr %LE %Long &value
        &rwp_true=1
        while &rwp_true==1
        (
                &rwp_true=D.Long(AD:&arg_gicd_ctlr)
                &rwp_true=&rwp_true>>31.
        )

        RETURN
GICV3_RDISTIF_INIT:
        ENTRY &arg_gicr_ctlr &arg_num_of_reg
        LOCAL &gicr_pwrr &pwrr_offset
        LOCAL &gicr_icenabler &enable_clr_offset
        LOCAL &rwp_true
        LOCAL &index

        &pwrr_offset=0x24
        &enable_clr_offset=&sgi_offset+0x180

        // Redistributor Power On //
        &gicr_pwrr=&arg_gicr_ctlr+&pwrr_offset
        D.S AD:&gicr_pwrr %LE %Long 0x0

        // All clear enable registers //
        &index=1.
        &gicr_icenabler=&arg_gicr_ctlr+&enable_clr_offset
        D.S AD:&gicr_icenabler %LE %Long 0xFFFFFFFF
        &rwp_true=1
        while &rwp_true==1
        (
                &rwp_true=D.Long(AD:&arg_gicr_ctlr)
                &rwp_true=&rwp_true>>31.
        )

        // Set Groups G1NS as default //
        &gicr_igroupr=&arg_gicr_ctlr+&sgi_offset+0x80
        D.S AD:&gicr_igroupr %LE %Long 0xFFFFFFFF

        RETURN

        GICV3_CPUIF_INIT:
        ENTRY &arg_gicr_ctlr
        LOCAL &gicr_waker
        LOCAL &icc_sre_el1 &icc_sre_el3
        LOCAL &icc_pmr_el1
        LOCAL &ca_bit

        &gicr_waker=&arg_gicr_ctlr+0x14

        // gicr core awake //
        &ca_bit=0.
        while &ca_bit==0
        (
                &ca_bit=D.Long(AD:&gicr_waker)
                &ca_bit=&ca_bit&0x7
                &ca_bit=&ca_bit>>2.
        )
        // clear PS bit and wait till ca_bit is 0 //
        D.S AD:&gicr_waker %LE %Long 0x4
        while &ca_bit!=0
                &ca_bit=D.Long(AD:&gicr_waker)

        // set SRE by exception levels //
        &icc_sre_el1=0x30CC5
        &icc_sre_el3=0x36CC5
        D.S SPR:&icc_sre_el3 %Quad 0xF ; auto set 0x7 in icc_sre_el1
        //////////////////////////////////////
        //          <pseudo code>           //
        // scr_el3 <- SCR_NS_BIT            //
        // icc_sre_el2 <- read(icc_sre_el3) //
        // scr_el3 <- ~SCR_NS_BIT           //
        // icc_sre_el1 <- 0x7               //
        //////////////////////////////////////

        // set pmr //
        &icc_pmr_el1=0x30460
        D.S SPR:&icc_pmr_el1 %Quad 0xF0 ; num of priority level: 16

        // set group enable for virtual IRQ //
        &icc_igrpen0_el1=0x30CC6
        D.S SPR:&icc_igrpen0_el1 %Quad 0x1
        &icc_igrpen1_el1=0x30CC7
        D.S SPR:&icc_igrpen1_el1 %Quad 0x1
        &icc_igrpen1_el3=0x36CC7
        D.S SPR:&icc_igrpen1_el3 %Quad 0x3

        RETURN
  </code></pre>
사용자는 User Configuration 파트만 수정해주면 된다.<br>

<B>User Configuration</B><br>
gicd_base: gicd base address를 작성한다.<br>
gicr_base: gicr core 0 lpi base address를 작성한다.<br>
gicr_num_of_regs: Chip boot 직후는 core 0 만 켜져있기 때문에 거의 1 고정<br>
  <p>
</details>
<br>

<details>
  <summary><B>GIC PPIs SPIs 테스트 스크립트</B></summary>
  <p>
  <pre><code>
/////////////////////////////
// gicv3 test script
//
// - delete WDT kernel configs
// B:: sys.config.gicd type gic600
// B:: sys.config.gicd base a:{GICD base address}
// B:: sys.config.gicr type a:{GICR base address}
// B:: per.reprogram
/////////////////////////////

&main=0x0
&sub=0x1

////////////////////////
// User Configuration //
////////////////////////
&num_of_chip=
&cluster=&main ; &main or &sub
&gicd_base_addr=
&gicr_base_addr=
&in_el3=0.
&eoi_mode=0.

//////////////////////////
// User Configuration 2 //
//////////////////////////
&start_irq=16. ; PPIs ~
&gic_max_irq=512.+1. ; 480(SPIs) + 32(SGIs, PPIs)

//////////////////////////
// User Configuration 3 //
//////////////////////////
&gic_target_core=0.


////////////////////////////
// ARM-A System Registers //
// addr prefix: "spr"     //
////////////////////////////
// EL3 //
&scr_el3=0x36110
// GIC CPU Interfaces //
&icc_iar0_el1=0x30c80
&icc_iar1_el1=0x30cc0
&icc_hppir0_el1=0x30c82
&icc_hppir1_el1=0x30cc2
&icc_eoir0_el1=0x30c81
&icc_eoir1_el1=0x30cc1
&icc_dir_el1=0x30cb1
&icc_sgi0r_el1=0x30cb7
&icc_sgi1r_el1=0x30cb5
// System Counter //
&cntp_ctl_el0=0x33e21
&cntv_ctl_el0=0x33e31
&cntps_ctl_el1=0x37e21
&cnthp_ctl_el2=0x34e20
&cntkctl_el1=0x30e10
&cnthctl_el2=0x34e10

///////////////////
// GIC registers //
///////////////////
// GIC offsets //
&iroute_offset=0x6100
&enable_offset=0x100
&enable_clr_offset=0x180
&grp_offset=0x80
&grpmod_offset=0xd00
&pend_offset=0x200
&pend_clr_offset=0x280
&active_offset=0x300
&active_clr_offset=0x380
&sgi_offset=0x10000
&num_of_reg=((&active_clr_offset-&active_offset)/4)+1
&num_of_spi=(&gic_max_irq-32.)
// GIC interrupt registers //
Var.NEWLOCAL long [&num_of_reg] \gic_isenabler
Var.NEWLOCAL long [&num_of_reg] \gic_icenabler
Var.NEWLOCAL long [&num_of_reg] \gic_groupr
Var.NEWLOCAL long [&num_of_reg] \gic_grpmodr
Var.NEWLOCAL long [&num_of_reg] \gic_ispendr
Var.NEWLOCAL long [&num_of_reg] \gic_icpendr
Var.NEWLOCAL long [&num_of_reg] \gic_isactiver
Var.NEWLOCAL long [&num_of_reg] \gic_icactiver
// GICD //
&gicd_base_addr=&gicd_base_addr+(&cluster*0x20000)
&gicd_route_base_addr=&gicd_base_addr+&iroute_offset
&gicd_enable_base_addr=&gicd_base_addr+&enable_offset
&gicd_enable_clr_base_addr=&gicd_base_addr+&enable_clr_offset
&gicd_grp_base_addr=&gicd_base_addr+&grp_offset
&gicd_grpmod_base_addr=&gicd_base_addr+&grpmod_offset
&gicd_pend_base_addr=&gicd_base_addr+&pend_offset
&gicd_pend_clr_base_addr=&gicd_base_addr+&pend_clr_offset
&gicd_active_base_addr=&gicd_base_addr+&active_offset
&gicd_active_clr_base_addr=&gicd_base_addr+&active_clr_offset
// GICR //
&gicr_base_addr=&gicr_base_addr+(&cluster*0x20000)+(&gic_target_core*0x20000)
&gicr_lpi_base_addr=&gicr_base_addr
&gicr_sgi_base_addr=&gicr_base_addr+&sgi_offset
&gicr_isenabler=&gicr_sgi_base_addr+&enable_offset
&gicr_icenabler=&gicr_sgi_base_addr+&enable_clr_offset
&gicr_groupr=&gicr_sgi_base_addr+&grp_offset
&gicr_grpmodr=&gicr_sgi_base_addr+&grpmod_offset
&gicr_ispendr=&gicr_sgi_base_addr+&pend_offset
&gicr_icpendr=&gicr_sgi_base_addr+&pend_clr_offset
&gicr_isactiver=&gicr_sgi_base_addr+&active_offset
&gicr_icactiver=&gicr_sgi_base_addr+&active_clr_offset

//////////
// Etc. //
//////////
// common //
&addr=0x0
&index=0.
&irq=0.
// group //
Var.NEWLOCAL long [&gic_max_irq] \irq_group
&irq_group_flag=1.


GOSUB ENV_PRESET

// GIC Verification //
GOSUB GIC_SETUP
GOSUB GIC_STATUS_INIT
GOSUB GIC_SET_AFFINITY &gic_target_core

sys.log.ON
&irq=&start_irq
WHILE &irq<&gic_max_irq
(
	//////////////////////////////////////////////////
	// Use '> continue' in script debug mode        //
	// <---------------------- (Break.Set)          //
	//////////////////////////////////////////////////
	GOSUB GIC_IRQ_HANDLE &irq
	&irq=&irq+1.
)
sys.log.OFF

ENDDO


ENV_PRESET:
	LOCAL &cpsr
	LOCAL &cpsr_nzcv &cpsr_endian &cpsr_async &cpsr_int &cpsr_mode
	LOCAL &scr
	LOCAL &scr_irq &scr_fiq &scr_ns

	Core.select &gic_target_core

	&cpsr=0x0
	&cpsr_nzcv=(0x4<<28.)
	&cpsr_endian=(0x1<<9.)
	&cpsr_async=(0x1<<8.)
	&cpsr_int=(0x3<<6.)
	&cpsr_mode=0xd ; EL3h

	&secure=0x0
	&scr_irq=(0x1<<1.)
	&scr_fiq=(0x1<<2.)
	&scr_ns=(0x1<<0.)

	&cpsr=&cpsr_nzcv|&cpsr_endian|&cpsr_async|&cpsr_int|&cpsr_mode
	Register.Set CPSR &cpsr

	&secure=D.Long(SPR:&scr_el3)
	&secure=&secure|&scr_irq|&scr_fiq
	&secure=&secure&(~&scr_ns)
	Data.Set SPR:&scr_el3 &secure

	// EL3 -> EL1 //
	if &in_el3==0
	(
		&secure=D.Long(SPR:&scr_el3)
		&secure=&secure&(~&scr_irq)
		&secure=&secure&(~&scr_fiq)
		Data.Set SPR:&scr_el3 &secure

		&cpsr_mode=0x5 ; EL1h
		&cpsr=&cpsr&0xFFFFFFF0
		&cpsr=&cpsr|&cpsr_mode
		Register.Set CPSR &cpsr
	)

	RETURN


GIC_REG_SETUP:
	&index=0.
	WHILE &index<&num_of_reg
	(
		if &index==0.
		(
			Var.set \gic_isenabler[&index]=&gicr_isenabler
			Var.set \gic_icenabler[&index]=&gicr_icenabler
			Var.set \gic_groupr[&index]=&gicr_groupr
			Var.set \gic_grpmodr[&index]=&gicr_grpmodr
			Var.set \gic_ispendr[&index]=&gicr_ispendr
			Var.set \gic_icpendr[&index]=&gicr_icpendr
			Var.set \gic_isactiver[&index]=&gicr_isactiver
			Var.set \gic_icactiver[&index]=&gicr_icactiver
		)
		else
		(
			Var.set \gic_isenabler[&index]=&gicd_enable_base_addr+(&index*0x4)
			Var.set \gic_icenabler[&index]=&gicd_enable_clr_base_addr+(&index*0x4)
			Var.set \gic_groupr[&index]=&gicd_grp_base_addr+(&index*0x4)
			Var.set \gic_grpmodr[&index]=&gicd_grpmod_base_addr+(&index*0x4)
			Var.set \gic_ispendr[&index]=&gicd_pend_base_addr+(&index*0x4)
			Var.set \gic_icpendr[&index]=&gicd_pend_clr_base_addr+(&index*0x4)
			Var.set \gic_isactiver[&index]=&gicd_active_base_addr+(&index*0x4)
			Var.set \gic_icactiver[&index]=&gicd_active_clr_base_addr+(&index*0x4)
		)
		&index=&index+1.
	)

	RETURN


EOI_MODE_SETUP:
	ENTRY &arg_mode
	LOCAL &icc_ctlr_el1 &icc_ctlr_el3
	LOCAL &eoi_mode_el1 &eoi_mode_el3

	if &arg_mode==0
	(
		&icc_ctlr_el1=0x30cc4
		&eoi_mode_el1=D.Long(SPR:&icc_ctlr_el1)
		&eoi_mode_el1=&eoi_mode_el1&(~0x2)
		PER.Set.simple SPR:&icc_ctlr_el1 %Quad &eoi_mode_el1
		if &in_el3==1.
		(
			&icc_ctlr_el3=0x36cc4
			&eoi_mode_el3=D.Long(SPR:&icc_ctlr_el3)
			&eoi_mode_el3=&eoi_mode_el1&(~0x10)
			PER.Set.simple SPR:&icc_ctlr_el3 %Quad &eoi_mode_el3
		)
	)

	RETURN


IRQ_GROUP_SETUP: // GICD_CTLR.DS == 0
	ENTRY &arg_id &arg_index
	LOCAL &val_groupr &val_grpmodr

	&val_groupr=D.Long(ASD:Var.value(\gic_groupr[&arg_index]))
	&val_groupr=&val_groupr>>(&arg_id%32.)
	&val_groupr=&val_groupr&0x1
	&val_grpmodr=D.Long(ASD:Var.value(\gic_grpmodr[&arg_index]))
	&val_grpmodr=&val_grpmodr>>(&arg_id%32.)
	&val_grpmodr=&val_grpmodr&0x1

	if (&val_groupr|&val_grpmodr)==1
		Var.set \irq_group[&arg_id]=1.
	else
		Var.set \irq_group[&arg_id]=0.

	RETURN


DS_IRQ_GROUP_SETUP: // GICD_CTLR.DS == 1
	ENTRY &arg_id &arg_index
	LOCAL &val_groupr

	&val_groupr=D.Long(ASD:Var.value(\gic_groupr[&arg_index]))
	&val_groupr=&val_groupr>>(&arg_id%32.)
	&val_groupr=&val_groupr&0x1

	if &val_groupr==1
		Var.set \irq_group[&arg_id]=1.
	else
		Var.set \irq_group[&arg_id]=0.

	RETURN


GIC_SETUP:
	LOCAL &gicd_ctlr &gicd_ctlr_ds

	GOSUB EOI_MODE_SETUP &eoi_mode
	GOSUB GIC_REG_SETUP

	&gicd_ctlr=D.Long(ASD:&gicd_base_addr)
	&gicd_ctlr=&gicd_ctlr&(1<<6)
	&gicd_ctlr_ds=&gicd_ctlr>>6
	if &gicd_ctlr_ds==0
	(
		&index=0.
		&id=0.
		WHILE &id<&gic_max_irq
		(
			GOSUB IRQ_GROUP_SETUP &id &index
			&id=&id+1.
			&index=(&id/32.)
		)
	)
	else
	(
		&index=0.
		&id=0.
		WHILE &id<&gic_max_irq
		(
			GOSUB DS_IRQ_GROUP_SETUP &id &index
			&id=id+1.
			&index=(&id/32.)
		)
	)

	RETURN


GIC_SET_AFFINITY:
	ENTRY &arg_target
	LOCAL &aff

	&aff=(&arg_target<<8.)
	Data.Set ASD:&gicd_route_base_addr++(&num_of_spi*8) %LE %Long &aff

	RETURN


GIC_SET_IRQ:
	ENTRY &arg_id

	Data.Set ASD:Var.Value(\gic_isenabler[&arg_id/32.]) %LE %Long (1<<(&arg_id%32.))
	Data.Set ASD:Var.Value(\gic_ispendr[&arg_id/32.]) %LE %Long (1<<(&arg_id%32.))

	RETURN


GIC_CLR_IRQ:
	ENTRY &arg_id

	Data.Set ASD:Var.Value(\gic_icpendr[&arg_id/32.]) %LE %Long (1<<(&arg_id%32.))
	Data.Set ASD:Var.Value(\gic_icenabler[&arg_id/32.]) %LE %Long (1<<(&arg_id%32.))

	RETURN


GIC_IRQ_HANDLE:
	ENTRY &arg_irq
	LOCAL &iar &restore_iar
	LOCAL &icc_iar &icc_eoi
	LOCAL &val_enabler &val_pendr &val_activer
	LOCAL &special_id

	if &arg_irq==32.
		GOSUB OFF_SYS_COUNTER &gic_target_core

	&irq_group_flag=Var.value(\irq_group[&arg_irq])
	&special_id=0.
	&spurious_cnt=0.

	if &irq_group_flag==0.
	(
		&icc_iar=&icc_iar0_el1
		&icc_eoi=&icc_eoir0_el1
	)
	else
	(
		&icc_iar=&icc_iar1_el1
		&icc_eoi=&icc_eoir1_el1
	)

	GOSUB GIC_SET_IRQ &arg_irq

	&iar=D.Long(SPR:&icc_iar)

	if (&iar&(~0x3))==1020.
		&special_id=1
	else
		&special_id=0

	if &special_id==0
	(
		// Check active status //
		&val_activer=D.Long(ASD:Var.Value(\gic_isactiver[&iar/32.]))
		&val_activer=&val_activer>>(&iar%32.)
		&val_activer=&val_activer&0x1

		// Check interrupt life-cycle pending -> active //
		if &val_activer==1
		(
			PRINT "Pass interrupt active irq: &iar"
			GOSUB GIC_CLR_IRQ &iar

			PER.Set.simple SPR:&icc_eoi %Quad &iar

			// set dir (eoi mode 1) //
			if &eoi_mode==1
				PER.Set.simple SPR:&icc_dir_el1 %Quad &iar

			// Check active status //
			&val_activer=D.Long(ASD:Var.Value(\gic_isactiver[&iar/32.]))
			&val_activer=&val_activer>>(&iar%32.)
			&val_activer=&val_activer&0x1

			// Check interrupt life-cycle active -> end //
			if &val_activer==0
				PRINT "Pass interrupt handle irq: &iar"
			else
				PRINT "Fail interrupt handle irq: &iar"
		)
		else
		(
			PRINT "Fail interrupt active irq: &iar"
			GOSUB GIC_CLR_IRQ &iar
		)
	)
	else
	(
		PRINT "Spurious interrupt! Check GIC status and irq group"
		PRINT "Recommand RESET and SYStem.Down"
		ENDDO
	)

	RETURN


GIC_STATUS_INIT:
	&index=0.
	WHILE &index<&num_of_reg
	(
		Data.Set ASD:Var.Value(\gic_icenabler[&index]) %LE %Long 0xFFFFFFFF
		Data.Set ASD:Var.Value(\gic_icpendr[&index]) %LE %Long 0xFFFFFFFF
		Data.Set ASD:Var.Value(\gic_icactiver[&index]) %LE %Long 0xFFFFFFFF
		&index=&index+1.
	)

	RETURN


OFF_SYS_COUNTER:
	ENTRY &arg_chip

	Core.select &arg_chip

	Data.Set spr:&cntp_ctl_el0 %LE %Long 0x2
	Data.Set spr:&cntv_ctl_el0 %LE %Long 0x2
	Data.Set spr:&cntps_ctl_el1 %LE %Long 0x2
	Data.Set spr:&cnthp_ctl_el2 %LE %Long 0x2

	Data.Set spr:&cntkctl_el1 %LE %Long 0x3
	Data.Set spr:&cnthctl_el2 %LE %Long 0x303

	RETURN
  </code></pre>
<br>

사용자는 User Configuration 파트만 수정해주면 된다.<br>

<B>User Configuration</B><br>
num_of_chip: 디버거 사용 타겟 칩의 최대 core 수를 작성한다.<br>
cluster: 멀티 클러스터 구조가 아니라면 main이 디폴트 값이다.<br>
gicd_base_addr: gicd의 시작 주소를 작성한다.<br>
gicr_base_addr: gicr core0 lpi 시작 주소를 작성한다.<br>
in_el3: el3 환경에서 테스트하면 1. 작성, el1 환경에서 테스트하면 0. 을 작성한다.<br>
eoi_mode: eoi mode 0 이면 0., 1이면 1. <br>
<br>

<B>User Configuration 2</B><br>
start_irq: 테스트 시작 인터럽트 번호를 작성한다. 보통은 PPIs 시작주소인 16을 쓴다.<br>
gic_max_irq: SoC에서 제공하는 SPI 인터럽트 최대 수를 작성한다. +1. 은 수정하지 않는다.<br>

<B>User Configuration 3</B><br>
gic_target_core: 인터럽트 테스트할 타겟 cpu 번호를 작성한다. num_of_chip값 보다 크면 안된다.<br>
  </p>
</details>
<br>

<details>
  <summary><B>GIC SGIs GENERATE & ROUTING 테스트 스크립트</B></summary>
  <p>
  <pre><code>
////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
// gicv3 sgi generate script
//
// - kernel 6.1 or later
// - delete WDT kernel configs
//
// - As of T32 Debugger 2023,
//   there is a difference between triggering SGI generation through step execution and using debugger commands to write.
//   Since verification is conducted through step execution, it is essential to load the Linux ELF before running this script.
//
////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

&main=0x0
&sub=0x1

////////////////////////
// User Configuration //
////////////////////////
&num_of_chip=8.
&cluster=&main
&gicd_base_addr=
&gicr_base_addr=
&in_el3=0.
&eoi_mode=0.

//////////////////////////
// User Configuration 2 //
//////////////////////////
&remote_storage=1.
&vmlinux="Z:\work1\tcc807x\main\kernel-6.1\vmlinux"
&invalid_part="/home/user"
&correct_part="Z:"

//////////////////////////
// User Configuration 3 //
//////////////////////////
&sgi_create_core=0.
&sgi_target_core=1.

////////////////////////////
// ARM-A System Registers //
// addr prefix: "spr"     //
////////////////////////////
// EL3 //
&scr_el3=0x36110
// GIC CPU Interfaces //
&icc_iar0_el1=0x30c80
&icc_iar1_el1=0x30cc0
&icc_hppir0_el1=0x30c82
&icc_hppir1_el1=0x30cc2
&icc_eoir0_el1=0x30c81
&icc_eoir1_el1=0x30cc1
&icc_dir_el1=0x30cb1
&icc_sgi0r_el1=0x30cb7
&icc_sgi1r_el1=0x30cb5
// System Counter //
&cntp_ctl_el0=0x33e21
&cntv_ctl_el0=0x33e31
&cntps_ctl_el1=0x37e21
&cntkctl_el1=0x30e10
&cnthctl_el2=0x34e10

///////////////////
// GIC registers //
///////////////////
// GIC offsets //
&enable_offset=0x100
&enable_clr_offset=0x180
&grp_offset=0x80
&grpmod_offset=0xd00
&pend_offset=0x200
&pend_clr_offset=0x280
&active_offset=0x300
&active_clr_offset=0x380
&sgi_offset=0x10000
// GICR //
&gicr_base_addr=&gicr_base_addr+(&cluster*0x20000)+(&sgi_target_core*0x20000)
&gicr_lpi_base_addr=&gicr_base_addr
&gicr_sgi_base_addr=&gicr_base_addr+&sgi_offset
&gicr_isenabler=&gicr_sgi_base_addr+&enable_offset
&gicr_icenabler=&gicr_sgi_base_addr+&enable_clr_offset
&gicr_groupr=&gicr_sgi_base_addr+&grp_offset
&gicr_grpmodr=&gicr_sgi_base_addr+&grpmod_offset
&gicr_ispendr=&gicr_sgi_base_addr+&pend_offset
&gicr_icpendr=&gicr_sgi_base_addr+&pend_clr_offset
&gicr_isactiver=&gicr_sgi_base_addr+&active_offset
&gicr_icactiver=&gicr_sgi_base_addr+&active_clr_offset

/////////////
// vmlinux //
/////////////

/////////////////////////////////////////////////////////////////////////////////////////////////////////
// diff --git a/drivers/irqchip/irq-gic-v3.c b/drivers/irqchip/irq-gic-v3.c                            //
// index 11f7c53e4b63..7d5937c375a8 100644                                                             //
// --- a/drivers/irqchip/irq-gic-v3.c                                                                  //
// +++ b/drivers/irqchip/irq-gic-v3.c                                                                  //
// @@ -1298,7 +1298,7 @@ static u16 gic_compute_target_list(int *base_cpu, const struct cpumask *mask, //
//         (MPIDR_AFFINITY_LEVEL(cluster_id, level) \                                                  //
//                 << ICC_SGI1R_AFFINITY_## level ##_SHIFT)                                            //
//                                                                                                     //
// -static void gic_send_sgi(u64 cluster_id, u16 tlist, unsigned int irq)                              //
// +void gic_send_sgi(u64 cluster_id, u16 tlist, unsigned int irq)                                     //
//  {                                                                                                  //
//         u64 val;                                                                                    //
/////////////////////////////////////////////////////////////////////////////////////////////////////////
&break_point=gic_send_sgi+0x40

//////////
// Etc. //
//////////
// common //
&addr=0x0
&core=0
&gic_max_irq=15.
&mpidr_el1=0x30005
&irq=0

GOSUB GIC_SGI_PRESET

sys.log.ON

// GIC SGI generate //
WHILE &irq<=&gic_max_irq
(
	//////////////////////////////////////////////////
	// Use '> continue' in script debug mode        //
	// <---------------------- (Break.Set)          //
	//////////////////////////////////////////////////

	GOSUB GIC_SGI_GENERATE &sgi_create_core &sgi_target_core &irq
	GOSUB GIC_SGI_HANDLE &sgi_target_core &irq
	&irq=&irq+1.
)

sys.log.OFF

ENDDO


GIC_SGI_PRESET:
	Data.LOAD.elf &vmlinux /NoCode
	if &remote_storage==1.
		sYmbol.sourcePATH.translate "&invalid_part" "&correct_part"

	WHILE &core<&num_of_chip
	(
		GOSUB OFF_SYS_COUNTER &core
		&core=&core+1.
	)

	Break.set &break_point

	RETURN


GIC_SGI_GENERATE:
	ENTRY &arg_dpt &arg_dst &arg_id
	LOCAL &aff1 &irq_id &tlist
	LOCAL &sgi_cmd &sgi_staus

	Core.select &arg_dst

	&irq_id=24.
	&aff1=16.
	&tlist=0x1
	&tlist=&tlist|D.Long(SPR:&mpidr_el1)&0xf

	Core.select &arg_dpt

	&sgi_cmd=(&arg_dst<<&aff1)
	&sgi_cmd=&sgi_cmd|(&arg_id<<&irq_id)
	&sgi_cmd=&sgi_cmd|&tlist

	Data.Set ASD:&gicr_isenabler %LE %Long (1<<&arg_id)
	Register.Set PC &break_point
	Register.Set X0 &sgi_cmd
	STEP

	&val_pendr=D.Long(ASD:&gicr_ispendr)>>&arg_id
	&val_pendr=&val_pendr&0x1

	if &val_pendr==1
		PRINT "Pass interrupt pending irq: &arg_id"
	else
		PRINT "Fail interrupt pending irq: &arg_id"

	RETURN


GIC_SET_IRQ:
	ENTRY &arg_id

	Data.Set ASD:&gicr_isenabler %LE %Long (1<<&arg_id)
	Data.Set ASD:&gicr_ispendr %LE %Long (1<<&arg_id)

	RETURN


GIC_CLR_IRQ:
	ENTRY &arg_id

	Data.Set ASD:&gicr_icenabler %LE %Long (1<<&arg_id)
	Data.Set ASD:&gicr_icpendr %LE %Long (1<<&arg_id)

	RETURN


GIC_SGI_HANDLE:
	ENTRY &arg_dst &arg_id

	Core.select &arg_dst
	&iar=D.Long(SPR:&icc_iar1_el1)
	// special ID: 1020 ~ 1023 //
	if (&iar&(~0x3))==1020.
		&special_id=1
	else
		&special_id=0

	if &special_id==0
	(
		&val_activer=D.Long(ASD:&gicr_isactiver)
		&val_activer=&val_activer>>&arg_id
		&val_activer=&val_activer&0x1

		if &val_activer==1
		(
			PRINT "Pass interrupt active irq: &iar"
			GOSUB GIC_CLR_IRQ &iar

			PER.Set.simple SPR:&icc_eoir1_el1 %Quad &iar

			// set dir (eoi mode 1) //
			if &eoi_mode==1.
				PER.Set.simple SPR:&icc_dir_el1 %Quad &iar

			&val_activer=D.Long(ASD:&gicr_isactiver)
			&val_activer=&val_activer>>&arg_id
			&val_activer=&val_activer&0x1

			// Check interrupt life-cycle active -> end //
			if &val_activer==0
			(
				PRINT "Pass interrupt handle irq: &iar"
				PRINT "Pass Core[&sgi_create_core] ===> Core[&sgi_target_core] [SGI:&iar]"
			)
			else
				PRINT "Fail interrupt handle irq: &iar"
		)
		else
		(
			PRINT "Fail interrupt active irq: &iar"
			GOSUB GIC_CLR_IRQ &iar
		)
	)
	else
	(
		PRINT "Spurious interrupt! Check GIC status and irq group"
		PRINT "Recommand RESET and SYStem.Down"
		ENDDO
	)

	RETURN


EOI_MODE_SETUP:
	ENTRY &arg_mode
	LOCAL &icc_ctlr_el1 &icc_ctlr_el3
	LOCAL &eoi_mode_el1 &eoi_mode_el3

	if &arg_mode==0
	(
		&icc_ctlr_el1=0x30cc4
		&eoi_mode_el1=D.Long(SPR:&icc_ctlr_el1)
		&eoi_mode_el1=&eoi_mode_el1&(~0x2)
		PER.Set.simple SPR:&icc_ctlr_el1 %Quad &eoi_mode_el1
		if &in_el3==1.
		(
			&icc_ctlr_el3=0x36cc4
			&eoi_mode_el3=D.Long(SPR:&icc_ctlr_el3)
			&eoi_mode_el3=&eoi_mode_el1&(~0x10)
			PER.Set.simple SPR:&icc_ctlr_el3 %Quad &eoi_mode_el3
		)
	)

	RETURN


GIC_RESET:
	ENTRY &arg_dst
	LOCAL &sgi_cenabler &sgi_cpendr &sgi_cactiver

	&sgi_cenabler=&gicr_icenabler+(0x20000*&arg_dst)
	&sgi_cpendr=&gicr_icpendr+(0x20000*&arg_dst)
	&sgi_cactiver=&gicr_icactiver+(0x20000*&arg_dst)

	Data.Set ASD:&sgi_cenabler %LE %Long 0xFFFFFFFF
	Data.Set ASD:&sgi_cpendr %LE %Long 0xFFFFFFFF
	Data.Set ASD:&sgi_cactiver %LE %Long 0xFFFFFFFF

	RETURN


OFF_SYS_COUNTER:
	ENTRY &arg_chip

	Core.select &arg_chip

	Data.Set spr:&cntp_ctl_el0 %LE %Long 0x2
	Data.Set spr:&cntv_ctl_el0 %LE %Long 0x2
	Data.Set spr:&cntps_ctl_el1 %LE %Long 0x2

	Data.Set spr:&cntkctl_el1 %LE %Long 0x3
	Data.Set spr:&cnthctl_el2 %LE %Long 0x303

	RETURN
  </code></pre>

사용자는 User Configuration 파트만 수정해주면 된다.<br>

<B>User Configuration</B><br>
num_of_chip: 디버거 사용 타겟 칩의 최대 core 수를 작성한다.<br>
cluster: 멀티 클러스터 구조가 아니라면 main이 디폴트 값이다.<br>
gicd_base_addr: gicd의 시작 주소를 작성한다.<br>
gicr_base_addr: gicr core0 lpi 시작 주소를 작성한다.<br>
in_el3: el3 환경에서 테스트하면 1. 작성, el1 환경에서 테스트하면 0. 을 작성한다.<br>
eoi_mode: eoi mode 0 이면 0., 1이면 1. <br>
<br>

<B>User Configuration 2</B><br>
remote_storage: nfs나 samaba 같은 remote 환경 사용 시, 1. 아니면 0.을 작성한다.<br>
vmlinux: remote_storage가 1일 경우, 빌드한 커널의 vmlinux 경로를 작성한다.<br>
invalid_part: remote_storage가 1일 경우, remote 입장에서의 변환이 필요한 경로를 작성한다.<br>
correct_part: remote_storage가 1일 경우, cmm script 실행 경로에서의 변환이 필요한 경로를 작성한다.<br>

<B>User Configuration 3</B><br>
&sgi_create_core: SGI를 트리거할 core 번호를 작성한다.<br>
&sgi_target_core: SGI를 처리할 core 번호를 작성한다.<br>
  </p>
</details>
<br>

