jobs:
    safety
    miscr-c2012
    strict-compiler
    build image


docker build image
test.sh
store artifacts output.text

DBC/CAN database
    BO_ 200 SENSOR_SONARS: 8 SENSOR
        bus_object msg_id msg_name: frame_length sender
    SG_ SENSOR_SONARS_mux M : 24|12@1+ (0.01,-20.48) [-20.48|20.47] "" DRIVER,IO
        signal name multiplexor : start|size@little_endian unsigned (scale/fraction, offset) [min|max] no_unit debugger
            min/max: 64-bit total, 12-bit, 0x1000 4096*0.01 - 20.48(40.96/2) = 20.47
    BA_ "FieldType" SG__500 IO_DEBUG_test_enum "IO_DEBUG_test_enum";
        attribute
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
    CAT_
        category definition
    CM_
        comment
    BU_
        bus unit/network nodes
    EV_
        env var
    REL_
        relation/node relation
    BS_
        bus speed/baudrate settings
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

time quanta 8 for 1mbps
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

bus0=can1 bus1=can2 bus2=can3 bus3/gmlan
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