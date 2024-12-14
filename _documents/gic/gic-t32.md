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

CPU만 켜져있는 bare metal 상태에서 GIC 전원 ON
```
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
```
<br>

GIC SGIs PPIs SPIs 테스트 스크립트
```
/////////////////////////////
// tcc807x gicv3 test script
//
// - delete WDT kernel configs
// - gic peri browser B:: per pergic.per 0x17600000
// - change el: el3 / el1h secure
/////////////////////////////

////////////////////////
// User Configuration //
////////////////////////
&core=0. ; 0 or 1 or 2 or 3 ...
&in_el3=0.
&gicd_base_addr={gicd base addr}
&gicr_base_addr={gicr base addr}+(&core*0x2000)
&sgi_offset=0x10000
&gicr_lpi_base_addr=&gicr_base_addr
&gicr_sgi_base_addr=&gicr_base_addr+&sgi_offset
&gic_max_irq={max irq num}+1. ;683(SPIs) + 32(SGIs, PPIs)
&eoi_mode=0.

///////////////////
// ARM configure //
///////////////////
// common //
&addr=0x0
&spi_start_id=32.
&index=0.
// EL3 //
&scr_el3=0x36110
// CPU Interfaces //
&icc_iar0_el1=0x30c80
&icc_iar1_el1=0x30cc0
&icc_hppir0_el1=0x30c82
&icc_hppir1_el1=0x30cc2
&icc_eoir0_el1=0x30c81
&icc_eoir1_el1=0x30cc1
&icc_dir_el1=0x30CB1
Var.NEWLOCAL long [&gic_max_irq] \irq_group
// gic offsets //
&enable_offset=0x100
&enable_clr_offset=0x180
&grp_offset=0x80
&grpmod_offset=0xd00
&pend_offset=0x200
&pend_clr_offset=0x280
&active_offset=0x300
&active_clr_offset=0x380
&num_of_reg=((&active_clr_offset-&active_offset)/4)+1

///////////////////
// GIC registers //
///////////////////

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
&gicd_enable_base_addr=&gicd_base_addr+&enable_offset
&gicd_enable_clr_base_addr=&gicd_base_addr+&enable_clr_offset
&gicd_grp_base_addr=&gicd_base_addr+&grp_offset
&gicd_grpmod_base_addr=&gicd_base_addr+&grpmod_offset
&gicd_pend_base_addr=&gicd_base_addr+&pend_offset
&gicd_pend_clr_base_addr=&gicd_base_addr+&pend_clr_offset
&gicd_active_base_addr=&gicd_base_addr+&active_offset
&gicd_active_clr_base_addr=&gicd_base_addr+&active_clr_offset
// GICR //
&gicr_isenabler=&gicr_sgi_base_addr+&enable_offset
&gicr_icenabler=&gicr_sgi_base_addr+&enable_clr_offset
&gicr_groupr=&gicr_sgi_base_addr+&grp_offset
&gicr_grpmodr=&gicr_sgi_base_addr+&grpmod_offset
&gicr_ispendr=&gicr_sgi_base_addr+&pend_offset
&gicr_icpendr=&gicr_sgi_base_addr+&pend_clr_offset
&gicr_isactiver=&gicr_sgi_base_addr+&active_offset
&gicr_icactiver=&gicr_sgi_base_addr+&active_clr_offset

GOSUB ENV_PRESET

sys.log.ON

// GIC Verification //
GOSUB GIC_SETUP
GOSUB GIC_TEST

sys.log.OFF

ENDDO

ENV_PRESET:
        LOCAL &cpsr
        LOCAL &cpsr_nzcv &cpsr_endian &cpsr_async &cpsr_int &cpsr_mode
        LOCAL &scr
        LOCAL &scr_irq &scr_fiq &scr_ns

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
        D.set SPR:&scr_el3 &secure

        // EL3 -> EL1 //
        if &in_el3==0
        (
                &secure=D.Long(SPR:&scr_el3)
                &secure=&secure&(~&scr_irq)
                &secure=&secure&(~&scr_fiq)
                D.set SPR:&scr_el3 &secure

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
        ENTRY &mode
        LOCAL &icc_ctlr_el1 &icc_ctlr_el3
        LOCAL &eoi_mode_el1 &eoi_mode_el3

        if &mode==0
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
        &val_groupr=&val_groupr&(1<<(&arg_id%32.))
        &val_groupr=&val_groupr>>(&arg_id%32.)
        &val_grpmodr=D.Long(ASD:Var.value(\gic_grpmodr[&arg_index]))
        &val_grpmodr=&val_grpmodr&(1<<(&arg_id%32.))
        &val_grpmodr=&val_grpmodr>>(&arg_id%32.)

        if (&val_groupr|&val_grpmodr)==1
                Var.set \irq_group[&arg_id]=1.
        else
                Var.set \irq_group[&arg_id]=0.

        RETURN


DS_IRQ_GROUP_SETUP: // GICD_CTLR.DS == 1
        ENTRY &arg_id &arg_index
        LOCAL &val_groupr

        &val_groupr=D.Long(ASD:Var.value(\gic_groupr[&arg_index]))
        &val_groupr=&val_groupr&(1<<(&arg_id%32.))
        &val_groupr=&val_groupr>>(&arg_id%32.)

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

        &index=0.
        WHILE &index<&num_of_reg
        (
                &index=&index+1.
        )

        RETURN

GIC_SET_IRQ:
        ENTRY &arg_id

        D.set ASD:Var.Value(\gic_isenabler[&arg_id/32.]) %LE %Long (1<<(&arg_id%32.))
        D.set ASD:Var.Value(\gic_ispendr[&arg_id/32.]) %LE %Long (1<<(&arg_id%32.))

        RETURN

GIC_CLR_IRQ:
        ENTRY &arg_id

        D.set ASD:Var.Value(\gic_icpendr[&arg_id/32.]) %LE %Long (1<<(&arg_id%32.))
        D.set ASD:Var.Value(\gic_icenabler[&arg_id/32.]) %LE %Long (1<<(&arg_id%32.))

        RETURN

GIC_TEST:
        LOCAL &iar &restore_iar &hppi
        LOCAL &irq_group_n &special_id
        LOCAL &val_pendr &val_activer
        LOCAL &irq_handled

        GOSUB GIC_RESET

        &irq_handled=0.
        &special_id=0.
        &irq_group_n=1.
        WHILE &irq_handled<&gic_max_irq
        (
                GOSUB GIC_SET_IRQ &irq_handled

                if &irq_group_n==0
                (
                        &hppi=D.Long(SPR:&icc_hppir0_el1)
                        &iar=D.Long(SPR:&icc_iar0_el1)
                        // special ID: 1020 ~ 1023 //
                        if (&iar&(~0x3))==1020.
                                &special_id=1
                        else
                                &special_id=0
                )
                else
                (
                        &hppi=D.Long(SPR:&icc_hppir1_el1)
                        &iar=D.Long(SPR:&icc_iar1_el1)
                        // special ID: 1020 ~ 1023 //
                        if (&iar&(~0x3))==1020.
                                &special_id=1
                        else
                                &special_id=0
                )

                if &special_id==0
                (
                        // Check groupr/grpmodr //
                        if &irq_group_n==Var.Value(\irq_group[&iar])
                                PRINT "Pass interrupt group irq: &iar"
                        else
                                PRINT "Fail interrupt group irq: &iar"

                        // Check active status //
                        &val_activer=D.Long(ASD:Var.Value(\gic_isactiver[&iar/32.]))
                        &val_activer=&val_activer&(1<<(&iar%32.))
                        &val_activer=&val_activer>>(&iar%32.)

                        //////////////////////////////////////////////////
                        // Use '> continue' in script debug mode        //
                        // <---------------------- (Break.Set)          //
                        //////////////////////////////////////////////////

                        // Check interrupt life-cycle pending -> active //
                        if &val_activer==1
                        (
                                PRINT "Pass interrupt active irq: &iar"
                                GOSUB GIC_CLR_IRQ &iar
                        )
                        else
                        (
                                PRINT "Fail interrupt active irq: &iar"
                                GOSUB GIC_CLR_IRQ &iar
                        )

                        // set eoi //
                        if &irq_group_n==0
                                PER.Set.simple SPR:&icc_eoir0_el1 %Quad &iar
                        else
                                PER.Set.simple SPR:&icc_eoir1_el1 %Quad &iar

                        // set dir (eoi mode 1) //
                        if &eoi_mode==1
                                PER.Set.simple SPR:&icc_dir_el1 %Quad &iar

                        // Check active status //
                        &val_activer=D.Long(ASD:Var.Value(\gic_isactiver[&iar/32.]))
                        &val_activer=&val_activer&(1<<(&iar%32.))
                        &val_activer_bit=&val_activer>>(&iar&32.)

                        // Check interrupt life-cycle active -> end //
                        if &val_activer==0
                        (
                                PRINT "Pass interrupt handle irq: &iar"
                                &irq_handled=&irq_handled+1.
                        )
                        else
                        (
                                PRINT "Fail interrupt handle irq: &iar"
                                &irq_handled=&irq_handled+1.
                        )
                )
                else if &special_id==1
                        &irq_group_n=&irq_group_n^1

                WAIT 20.MS
        )

        RETURN

GIC_RESET:
        D.set ASD:Var.Value(\gic_icenabler[0])++&num_of_reg-1 %LE %Long 0xFFFFFFFF
        D.set ASD:Var.Value(\gic_icpendr[0])++&num_of_reg-1 %LE %Long 0xFFFFFFFF
        D.set ASD:Var.Value(\gic_icactiver[0])++&num_of_reg-1 %LE %Long 0xFFFFFFFF

RETURN
```
<br>
