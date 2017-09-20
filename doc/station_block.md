```pascal
// Start example for a station blocks. A station block is a FB that is
// written for one specific modbus device.

// Every device block should start with a device header. Though it
// isn't required.
"mb_station_header"(station := #s, mb_query := #mb_query);


// Modbus station address (device address), same for all queries 
// in this station block.
#mb_query.mb_addr := #mb_addr;


// First query example
#mb_query(mode := #mb_query.c.read.holding_reg,
          data_addr := 123,           
          data_ptr := #resault_data_query_1);
//
// A standard modbus telegram
// .---------.--------.---------------.--------------.---  ..  ..  -- --.---------------.
// | mb_addr |  mode  |   data_addr   |    d_len     |      data_ptr    |      CRC      |
// '---------'--------'---------------'--------------'---- --  ..  ..  -'---------------'
//
// --- mb_addr:
// Station address. 1-247 (RTU device address)
//
// --- mode:
// Modbus function code, supported codes: 1, 2, 3, 4, 5, 6, 15 and 16. 
// Keep in mind that modes are shifted, e.g. fc 3 corresponds to mode 103.
// Start typing "#mb_query.c." and the auto complete list will show all 
// available function codes. 
//
// --- data_addr:
// Data address, the actual value that will be sent in the modbus telegram.
// Eg. to read holding address 123, you simply pass 123 to data_addr and
// "#mb_query.c.read.holding_reg" (103) to the mode-parameter. 40001 should NOT 
// be added to 123.
//
// --- data_len:
// Data length. Can be omitted and will then be calculate automatically. 
//
// --- data_ptr:
// Container for data that will be send or receive. The parameter is not
// limited to words, but can contain any data types, including: real,
// bytes, arrays and udt's
// Though, two criterion most to be followed:
// 1. Only whole registers. (Bytes and bool's should add up to 16 bits)
// 2. The modbus protocol limit each query to a maximum of 123 words.
//    (Sometimes up to 125 words)


// Similar query as above, except one input register are read. Any number
// of queries can be added, the library will take care of executing them,
// one by one.
#mb_query(mode := #mb_query.c.read.input_reg,
          data_addr := 1234,
          data_ptr := #data_query_2);


// Modicon convention addressing:
#mb_query(mode := #mb_query.c.mode.read,
          data_addr := 40005,           
          data_ptr := #data_query_3);
		   
// The parameters: mb_addr and data_ptr work the same way as in the 
// previous example. To use the Modicon convention addressing, one of the 
// mode-constants bellow should be used:
//   - #mb_query.c.mode.read           -> mb function code: 1, 2, 3 and 4  
//   - #mb_query.c.mode.write          -> mb function code: 15 and 16      
//   - #mb_query.c.mode.write_single   -> mb function code: 5 and 6        
// 
// When the Modicon convention is used, the function code is "embedded" in
// to data_addr. Eg. query 40005 will actually query holding register with
// data_ptr address (offset) 4. 


// A write query executed with function code "6" (write.single_holding_reg)
#mb_query(mode := #mb_query.c.write.single_holding_reg,
          data_addr := 42,           
          data_ptr := #data_ptr_query_4);
		   
// Some (old) modbus devices doesn't support modbus function code "16" In 
// that case function code number "6" need to be used 
// ("#mb_query.c.write.single_holding_reg"). 


// Header and footer
// -----------------------------------
// As mention at the top the mb_device_header isn't required However if
// the "footer" is implemented then the "header" is also needed, else the
// library will not work. No query should go before the "header" or after
// the "footer".

"mb_station_footer"(station := #s, mb_query := #mb_query);

// Benefits of the header and the fotter :
//  - Logging capacities: Inside the station-udt there is logging of
//    successfully and failed queries, which is helpful
//    for debugging. See: "#s.log. ..."
//  - "#s.out.error" a common error flag that is set if any of the
//    intermediate queries between mb_device_footer and mb_device_header
//    gives an error.
//  - "#s.out.error_comm" a error flag that will be set after number 
//    of repeating queries resulting in comm. error (timeout or bad crc)
//    of the intermediate queries. Intended
//    to reset values when communication problems occur. The number of
//    repeating timeout before the flag is set can be configured with
//    "#s.conf.max_repeating_comm_errors".
//  - Only one query for each device is allowed to be executed for
//    every loop. This is to avoid stations with many queries to occupy
//    the bus for long periods of time.
//  - Prevent the impact of setting if-condition around queries. The
//    "hiccup" effect mentioned above.


// For this example values are set to zero if communication problems accrue.
IF #common.out.communication_error THEN
    #resault_data_query_1 := 0;
    
    #data_query_2[1] := 0;
    #data_query_2[2] := 0;
END_IF;
```