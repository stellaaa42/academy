# openpilot panda opendbc notes

## DBC/CAN database container
    BO_ 200 SENSOR_SONARS: 8 SENSOR
        object id name: length sender
    SG_ SENSOR_SONARS_mux M : 
        24|12@1+ (0.01,-20.48) [-20.48|20.47] "" DRIVER,IO
            start|size@little_endian+/unsigned (scale/fraction, offset) [min|max] unit debugger
            min/max: 64-bit total, 12-bit, 0x1000 4096*0.01 - 20.48(40.96/2) = 20.47
        56|8@1+ (1,0) [0|255] ""  aps
    BA_ "FieldType" SG__500 IO_DEBUG_test_enum "IO_DEBUG_test_enum";
    multiplex 64-bit total: 
        2-bit 2^3=8 64-2=62 62*8=248 bytes
            8-bit 2^8=256 64-8=56 56*256=1792 bytes
        M m0/m1: two separate msgs for m0 and m1
    NS_
        new symbols
    BA_DEF
        attribute definition
    BA_DEF_DEF_
        attribute default value
    BO_
        bus object
    BS_
        bus speed/baudrate settings
    BU_
        bus unit/network nodes
    CAT_
        category definition
    CM_
        comment
    EV_
        env var
    REL_
        relation/node relation
    SG_
        signal
    agc -> auto generated code
        mia -> missing in action handling -> default temp=68f
    typedef struct {
        int8_t MOTOR_CMD_steer;
        uint8_t MOTOR_CMD_drive;
    } MOTOR_CMD_t;
        BO_ 101 MOTOR_CMD: 1 DRIVER
            SG_ MOTOR_CMD_steer : 0|4@1- (1, -5) [-5|5] "" MOTOR
            SG_ MOTOR_CMD_drive : 4|4@1- (1, 0) [0|9] "" MOTOR
    100hz -> 100 cycles/sec 1s=1000ms -> one cycle 1000/100=10ms

    big-endian 0x12 0x34 -> 0x1234
        little-endian -> 0x3412
    05 96 E7 54 6D 58 00 6F 
        05 (5, 0) scale by 5 5*5=25A current_limit
        96 E7 (0.25, 500) _6E7 hex6E7=1767 *0.25+500=58.25A battery_current
        6D 58 (0,0025, 0) hex6D58=27992 *0.0025=68.98% soc state_of_charge

## opendbc
    database 
        dbc format -> parse -> py
        steering
    mull: test mutation/programming logic
    dbc
        BU_: CH DIPF DIPR ETH FC HVI HVS PARTY SDCV VEH VIRT
            BO_ 905 DAS_status2: 8 PARTY
            SG_ DAS_status2Checksum : 56|8@1+ (1,0) [0|255] ""  aps
        party: ecu, sender
            das: driver assistance sys
            aps: accelerator pedal sensor
            esp: electronic stability programme
            gtw: gateway
    d = (dat[i] >> (lsb - (i * 8))) & ((1 << size) - 1)
        i=0, lsb-0=lsb, move lsb bits, 1<<2 2^2 0xFF, keep only the last two bit
        i=1, lsb-8, shift lsb-offset(align lsb to bit0) higher bits
        -> extract sig val from CAN msg
    @dataclass

## panda flow
### board/main.c
    irq init disable
    clk
    init peripheral
    board type
    init
        led for debug
        adc analog/voltage/current-to-digital converter
            adc peripheral
        can mode normal
        harness 
        fpu floating point unit/operation
        timer
        fan
            test power->rpm
            tuning: overshoot
                stop completely: sleep(0.1)
                30% power/onroad -> max rpm power 
                max / expected from panda
        safety mode silent
            clear msg hardware queue in can core/unplugged
            set obd diagnostics mode when param 0ms
        can tx transceiver
        heartbeat loop reset/feed watchdog 375ms/avoid hanging
        fault tick/timer 8hz irq 10ms   
            siren pulsing
                countdown

        usb
        spi
        irq
        led blink/fade increase, ON longer, OFF shorter, pulse smoothly
            pattern breaks -> sys overload 
    wfi wait for irq, lowe-power sleep till irq 
    scb_scr sys control block/register
        cpu sleep, not peripheral

### board/body/main.c
### board/body/can.h
    llcan low-level can
        llcan_init() reinit CAN at hardware level
    ring uart circular buffer
        data: uart inp
        loop: read data/char, inject back into ring
    acc: adaptive cruise control
    ul: unsigned long literal/32-bit
    u: unsigned integer literal suffix
    cpacr coprocessor access control register
        enable/disable 3 full access
        set bits[21:20] 3UL << (10U*2U)
    CAN periodic
        can command -> motor set target rpm left right
        if now - last >= period, send speed
    CAN init
        set gpio output
        safety hook
    CAN pkt
        bus-addr-returned
        rejected/extend/fd
        data_len
        data[0]...[7]
        checksum
        
### board/body/motor_controller.h
    inline funcs: speed_update()/write_pwm()
    absf: float
    control += 
        +kff feed-forward gain * target_rpm
        +kp proportional * error
        +ki integral
        +kd derivative
    motor write
        cfg config forward_timer forward_channel reverse_timer
        set gpio output
            enable port/pin

## board/body/flash.h
    scons .bin.signed
    parse arg
    get_usb_serial

## py __init__.py
    interact w/ openpilot
    can 
        congestion -> NAK
        reset comm
            controlWrite(type, req, index, data) abstractmethod
        send many
            pack buffer/header, len, bus, addr, extended -> append arr
            loop bulkwWrite tx
        receive
            bytearray bulkRead
            unpack
        clear
    version
    init
        dfupdate
        connect to serial
            usb fst, spi snd
            usb: serialnum, in usb_list
            spi: check bootstub pid
    reset
    reconn
    flash
        choose sectors accumulate enum
        unlock
        erase
        flash via ep2/bulkWrite
        reset controlWrite()
    recover
        dfu bootsub/program
    dfu timeout wait
        monotonic() not going back
        list
    wait for panda
        serial list
    signature up to date
    call/write control/handler api
    health
        struct
        uptime, voltage, current, buf overflow, ignition, harness, safety
    can health
        bus_off, error, last_error, total_rx tx lost cnt, speed, irq, core rst cnt
    self._handle.controlread rqt
        uid, secret, irq rate
    config
        alter, power, loopback, obd, ena, speed, uart baud/parity equal/callback
    serial
        read, write
        hearbeat send/disable
    timer
    ir power
    fan power set/get rpm unpack
    siren
    debug
        arr timer period
        ccr channel pulse len
        force relay
        read gpio
        
## debug
    openocd  open on-chip debugger
       .cfg stlink interface config
       swd serial wire debug protocol transport
       -f load stm32h7 config
       init

## dfu device firware update
    bootstub/recover
        clear/erase
        program() write mcu_type.config.addr, code
    usb/spi connect/list


## drivers/usb.h
    tx comm
    enter critical
    EP3 out endpoint3 not processing
    EP(3U)->DOEPCTL access out endpoint 3 registers -> control
    OTG->NAKSTS usb on-the-go/phone to usb directly -> NAK/refusing data status bit
    set SIZ size 32UL<<19 32 pkts(shift to bit 19 pos) of 0x800U 2048 bytes transfer size
    ENA enable to accept data | CNAK clear NAK/refusing data bit
    exit critical

    usb port
    irq handler
        gintsts global irq status reg
            flag & usbrst reset, mmism mode mismatch, enumdne enum done
        gccfg global core config reg
            power-down mode, vbus sensing
        cidschg id status change
        rxflvl receive FIFO level
            rxstsp pktsts read rx/pkt status 
        updt update 
            DATA, SETUP
            EP endpoint, BCNT len
        gnak global negtive ack 
            in, out
            dctl device control
        endpoint 
            in, out
        clear handled irq
    
    usb set_up
        hex 0x0040U
        device desc/version/len, type
        config
            len, type, index, class sub, endpoint, dir, max pkt, polling interval
        str language
        driver
        binary obj store
        enable other endpoints
            config, addr, req
    reset
        unmask endpoint irq
            DAINT dataline irq
            DAINTMSK
        clear irq
            DIEPINT, DOEPINT = 0xFF
        DCFG device config reg
            unset addr
        GRXFSIZ FIFO setup
        FLSH flush fifo tx rx
        no global NAK dctl
        ready to receive pkt DOEPTSIZ
    write pkt
        EP0 DIEPTSIZ DFIFO
    read pkt
        dest_copy++ dest 
        FIFO

## drivers/spi.h
    spi transfer msg struct
        header: endpoint, tx_len, rx_len, sync
        len
        tx_buf
        rx_buf
        speed_hz
    init &msg msg val/add tail
    memcpy header -> tx_buffer
    sync
    ack
        sync read spidev_data len
        == -> ret 0
        else -1
        >20 -> sleep
    response: rx_buf rx_len
    checksum
        ^= buf[0U, len++] xor accum
    retval copy -> user
    return rx_len

## stm32h7 config
    critical/faults/util
    driver registers/irqs/uart
    gpio/peripheral/irq/timers
    clock/usb/spi
    gpio:
        moder: mode register
            config pin mode/00 inp/01 general out/10 alternate/11 analog
        odr: out data register, out high/low
        pupdr: pull-up/down register internal resistor

## examples/tester.py
    adapter 
        powertrain bus0
        body bus1
        chassis bus2
    safety alloutput/careful!
    0x248 mcu cmd msg
        0x01 lock/pop frunk
        0x04 open trunk
    can send msg
    safety silent
    read vin(index+str)
        bytes -> hex str -> int
        [:2] first 2, [2:] the rest 

## examples/bit_transition.py
    msg low high range, start end
    transition low to high: parse, sort, diff
        after ':' hex->int
    csv file -> msg collection dict
        msg id: [2:] int
        bytearray data
            ones 
            ~ invert -> zeros
    transition
       zero_to_one bitmask: other.zeros[i] & self.ones[i]
       one_to_zero

## examples/query_fw_versions.py
    parser arg
        addr
        sub_addr
            'scan' 0x0-0xff
        uds unified diagnostic services data id: std_id
        serial panda
    query tqdm progress bar library/indicator
        bus
        sub_addrs
        description str to addr
        -> uds client tester session control
        id -> data -> resp{}, (addr, sub_addr) -> res[]

## math
### time quanta/split 8 for 1mbp
        1 megabits per sec when prescaler*8/clk = 1us
    bit time on CAN bus divided into series of quanta
     ~~quanta = (prescaler + 1)/clk~~
    prescaler/baudrate prescaler brp
        = clk*10/(quanta*bit_rate/can_speed)
        42mhz = 42e6/(8*1m) = 5.25 -> 5 
        48mhz = 48e6/(8*1m) = 6
        sample ptr 87.5% seq1 quanta 6 -> desired_ts1
                    12.5% seq2 quanta 1 -> desired_ts2
                6 + 1 + 1 = 8
    btr/bit time register /+1: hardware
        ~~= (1+ts1+1+ts2+1)*(prescaler+1)/clk~~
            #define CAN_BTR_TS1_0 0x00010000~~U~~ bit16 [19:16]
                *ts1/<<16
            #define CAN_BTR_TS2_0 0x00100000~~U~~ bit20 [22:20]
                *ts2/<<20
            prescaler 42mhz 5-1=4 [9:0] +1=5
        = (seq1-1)*ts1 | (seq2-1)*ts2 | prescaler-1
        = 5*0x00010000 = 1 left shift 5 times = bit[18:16] 101~~b~~ = 5+1=6 
            | 0*0x00100000 = bit[22:20] 0+1=1 
        = 0x001B0004_42mhz/0x001B0005_48mhz

### bus0=can1 bus1=can2 bus2=can3 bus3/gmlan
    can_speed[] = {5000, 5000, 5000, 333}
    logical bus bus_lookup[physical] = logical 
        bus_lookup[] = {0,1,2}
            bus_lookup[1/2] = 3: can1/2 map to bus3
    physical can: can_num_lookup[logic] = physical
        can_num_lookup[] = {0,1,2,-1}
        *cans[] = {CAN1, CAN2, CAN3}
            can_num_lookup[1/2] = -1: invalid, bus1/2
            can_num_lookup[3] = 1/2: gmlan take over can3
                bus3 for can1/2
            

        