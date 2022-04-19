`define HIGH 1'b1
`define LOW 1'b0
`define ADD 4'b0001
`define SUBTRACT 4'b0010
`define SLL 4'b0101
`define SRL 4'b0110
`define NO_OP 4'b0000
`define TIMEOUT 50
`define VAID_RESP 2'b01
`define DISPLAY 1

interface calc_interface (input a_clk, b_clk, c_clk, reset);
   wire [0:31] out_data1, out_data2, out_data3, out_data4;
   wire [0:1]  out_resp1, out_resp2, out_resp3, out_resp4, out_tag1, out_tag2, out_tag3, out_tag4;
   wire     scan_out;

   logic     scan_in;
   logic [0:3]      req1_cmd_in, req2_cmd_in, req3_cmd_in, req4_cmd_in;
   logic [0:1]      req1_tag_in, req2_tag_in, req3_tag_in, req4_tag_in;
   logic [0:31]  req1_data_in, req2_data_in, req3_data_in, req4_data_in;

    clocking cb @(posedge c_clk);
        default input #1 output #1;
        input req1_cmd_in, req1_data_in, req1_tag_in, req2_cmd_in, req2_data_in, req2_tag_in, req3_cmd_in, req3_data_in, req3_tag_in, req4_cmd_in, req4_data_in, req4_tag_in, reset, scan_in;
        output out_data1, out_data2, out_data3, out_data4, out_resp1, out_resp2, out_resp3, out_resp4, out_tag1, out_tag2, out_tag3, out_tag4, scan_out;
    endclocking
   
   modport DUT (output out_data1, out_data2, out_data3, out_data4, out_resp1, out_resp2, out_resp3, out_resp4, out_tag1, out_tag2, out_tag3, out_tag4, scan_out, input a_clk, b_clk, c_clk, req1_cmd_in, req1_data_in, req1_tag_in, req2_cmd_in, req2_data_in, req2_tag_in, req3_cmd_in, req3_data_in, req3_tag_in, req4_cmd_in, req4_data_in, req4_tag_in, reset, scan_in);
endinterface

module calc_tb;
    reg a_clk, b_clk, c_clk, reset;

    calc_interface calc_intf(a_clk, b_clk, c_clk, reset);

    calc2_top DUT (
        .out_data1(calc_intf.out_data1), 
        .out_data2(calc_intf.out_data2), 
        .out_data3(calc_intf.out_data3), 
        .out_data4(calc_intf.out_data4), 
        .out_resp1(calc_intf.out_resp1), 
        .out_resp2(calc_intf.out_resp2), 
        .out_resp3(calc_intf.out_resp3), 
        .out_resp4(calc_intf.out_resp4), 
        .out_tag1(calc_intf.out_tag1), 
        .out_tag2(calc_intf.out_tag2), 
        .out_tag3(calc_intf.out_tag3), 
        .out_tag4(calc_intf.out_tag4), 
        .scan_out(calc_intf.scan_out), 
        .a_clk(calc_intf.a_clk), 
        .b_clk(calc_intf.b_clk), 
        .c_clk(calc_intf.c_clk), 
        .req1_cmd_in(calc_intf.req1_cmd_in), 
        .req1_data_in(calc_intf.req1_data_in), 
        .req1_tag_in(calc_intf.req1_tag_in), 
        .req2_cmd_in(calc_intf.req2_cmd_in), 
        .req2_data_in(calc_intf.req2_data_in), 
        .req2_tag_in(calc_intf.req2_tag_in), 
        .req3_cmd_in(calc_intf.req3_cmd_in), 
        .req3_data_in(calc_intf.req3_data_in), 
        .req3_tag_in(calc_intf.req3_tag_in), 
        .req4_cmd_in(calc_intf.req4_cmd_in), 
        .req4_data_in(calc_intf.req4_data_in), 
        .req4_tag_in(calc_intf.req4_tag_in), 
        .reset(calc_intf.reset), 
        .scan_in(calc_intf.scan_in)
        );

    initial begin
        a_clk = `LOW;
        b_clk = `LOW;
        c_clk = `LOW;
        reset = `HIGH;
        forever begin
            fork
                begin #10 a_clk = ~a_clk; end
                begin #10 b_clk = ~b_clk; end
                begin #10 c_clk = ~c_clk; end
            join
        end
    end

    task initialize();
        calc_intf.req1_cmd_in = 32'b0;
        calc_intf.req1_data_in = 32'b0;
        calc_intf.req1_tag_in = 32'b0;
        calc_intf.req2_cmd_in = 32'b0;
        calc_intf.req2_data_in = 32'b0;
        calc_intf.req2_tag_in = 32'b0;
        calc_intf.req3_cmd_in = 32'b0;
        calc_intf.req3_data_in = 32'b0;
        calc_intf.req3_tag_in = 32'b0;
        calc_intf.req4_cmd_in = 32'b0;
        calc_intf.req4_data_in = 32'b0;
        calc_intf.req4_tag_in = 32'b0;
        calc_intf.scan_in = 32'b0;
    endtask

    task deassert_reset();
        repeat(5) @(posedge c_clk);
        reset = `LOW;
    endtask

    task driver_req1(input logic [0:3] cmd,logic [0:31] op1, logic [0:31] op2, logic [0:1] tag);
        @(posedge c_clk);
        //calc_intf.scan_in = `HIGH;
        calc_intf.req1_cmd_in = cmd;
        calc_intf.req1_data_in = op1;
        calc_intf.req1_tag_in = tag;
        @(posedge c_clk);
        calc_intf.req1_cmd_in = `NO_OP;
        calc_intf.req1_data_in = op2;
        calc_intf.req1_tag_in = `NO_OP;
        @(posedge c_clk);
        calc_intf.req1_data_in = 32'b0;
    endtask

    task monitor_req1(input logic [0:1] tag, output logic [0:1] resp, logic [0:31] data);
        @(posedge c_clk);
        wait(calc_intf.out_resp1 != 2'b00);
        wait(tag == calc_intf.out_tag1) begin
            resp = calc_intf.out_resp1;
            data = calc_intf.out_data1;
        end
        wait(calc_intf.out_resp1 == 2'b00);
        @(posedge c_clk);
    endtask

    task scoreboard (input logic [0:3] cmd,logic [0:31] op1, logic [0:31] op2, logic [0:1] resp, logic [0:31] data, output logic status);
        bit [31:0] result;
        case (cmd)
            `ADD : begin result = op1 + op2; end
            `SUBTRACT : begin result = op1 - op2; end
            `SRL : begin result = op1 >> op2; end
            `SLL : begin result = op1 << op2; end
            default : result = 32'b0;
        endcase

        if(resp == `VAID_RESP) begin
            if (data == result)
            begin
                if(`DISPLAY)
                $display("[T = %0t] - Transaction Passed : - Command %b,  op1:%h, op2:%h, Expected output:%h, Received Output:%h", $realtime, cmd,op1,op2,result,data); 
                status = 0; 
            end
            else
            begin
                if(`DISPLAY)
                $display("[T = %0t] - Transaction Failed : Wrong Output - Command %b, op1:%h, op2:%h, Expected output:%h, Received Output:%h", $realtime,cmd,op1,op2,result,data); 
                status = 1; 
            end
        end
        else begin
            if(`DISPLAY)
            $display("[T = %0t] - Transaction Failed: Error in Command / Overflow / Underflow - RESP:%h,Command %b, op1:%h, op2:%h, Expected output:%h, Received Output:%h", $realtime,resp,cmd,op1,op2,result,data); 
            if(result!=data)status = 1; 
        end
    endtask

    task driver_req3(input logic [0:3] cmd,logic [0:31] op1, logic [0:31] op2, logic [0:1] tag);
        @(posedge c_clk);
        calc_intf.scan_in = `HIGH;
        calc_intf.req3_cmd_in = cmd;
        calc_intf.req3_data_in = op1;
        calc_intf.req3_tag_in = tag;
        @(posedge c_clk);
        calc_intf.req3_cmd_in = `NO_OP;
        calc_intf.req3_data_in = op2;
        calc_intf.req3_tag_in = `NO_OP;
        @(posedge c_clk);
        calc_intf.req3_data_in = 32'b0;
    endtask

    task monitor_req3(input logic [0:1] tag, output logic [0:1] resp, logic [0:31] data);
        @(posedge c_clk);
        wait(calc_intf.out_resp3 != 2'b00);
        wait(tag == calc_intf.out_tag3) begin
            resp = calc_intf.out_resp3;
            data = calc_intf.out_data3;
        end
        wait(calc_intf.out_resp3 == 2'b00);
        @(posedge c_clk);
    endtask

    task driver_req2(input logic [0:3] cmd,logic [0:31] op1, logic [0:31] op2, logic [0:1] tag);
        @(posedge c_clk);
        calc_intf.scan_in = `HIGH;
        calc_intf.req2_cmd_in = cmd;
        calc_intf.req2_data_in = op1;
        calc_intf.req2_tag_in = tag;
        @(posedge c_clk);
        calc_intf.req2_cmd_in = `NO_OP;
        calc_intf.req2_data_in = op2;
        calc_intf.req2_tag_in = `NO_OP;
        @(posedge c_clk);
        calc_intf.req2_data_in = 32'b0;
    endtask

    task monitor_req2(input logic [0:1] tag, output logic [0:1] resp, logic [0:31] data);
        @(posedge c_clk);
        wait(calc_intf.out_resp2 != 2'b00);
        wait(tag == calc_intf.out_tag2) begin
            resp = calc_intf.out_resp2;
            data = calc_intf.out_data2;
        end
        wait(calc_intf.out_resp2 == 2'b00);
        @(posedge c_clk);
    endtask

    task driver_req4(input logic [0:3] cmd,logic [0:31] op1, logic [0:31] op2, logic [0:1] tag);
        @(posedge c_clk);
        calc_intf.scan_in = `HIGH;
        calc_intf.req4_cmd_in = cmd;
        calc_intf.req4_data_in = op1;
        calc_intf.req4_tag_in = tag;
        @(posedge c_clk);
        calc_intf.req4_cmd_in = `NO_OP;
        calc_intf.req4_data_in = op2;
        calc_intf.req4_tag_in = `NO_OP;
        @(posedge c_clk);
        calc_intf.req4_data_in = 32'b0;
    endtask

    task monitor_req4(input logic [0:1] tag, output logic [0:1] resp, logic [0:31] data);
        @(posedge c_clk);
        wait(calc_intf.out_resp4 != 2'b00);
        wait(tag == calc_intf.out_tag4) begin
            resp = calc_intf.out_resp4;
            data = calc_intf.out_data4;
        end
        wait(calc_intf.out_resp4 == 2'b00);
        @(posedge c_clk);
    endtask

    task operation_with_reset_HIGH();
        logic [0:3] resp_received;
        initialize;
        resp_received = 4'd0;
        
        $display("---------- STARTING TEST: operation_with_reset_HIGH ----------");
        reset = `HIGH;
        repeat(2)@(posedge c_clk);
        driver_req1(`ADD,32'd15,32'd5,2'b00);
        driver_req2(`SUBTRACT,32'd15,32'd5,2'b00);
        driver_req3(`SLL,32'd15,32'd5,2'b00);
        driver_req4(`SRL,32'd15,32'd5,2'b00);

        fork
            begin wait(calc_intf.out_resp1 != 2'b00); resp_received[0] = `HIGH;  end
            begin wait(calc_intf.out_resp2 != 2'b00); resp_received[1] = `HIGH; end
            begin wait(calc_intf.out_resp3 != 2'b00); resp_received[2] = `HIGH; end
            begin wait(calc_intf.out_resp4 != 2'b00); resp_received[3] = `HIGH; end
            begin repeat(`TIMEOUT) @(posedge c_clk); end
        join_any

        if(resp_received == 4'b0)
            $display("TEST PASSED - 'operation_with_reset_HIGH'");
        else
            $display("TEST FAILED - 'operation_with_reset_HIGH'");
        
        deassert_reset();
        $display("---------- TEST END: operation_with_reset_HIGH ----------");
    endtask

    task operations_on_req1_without_overflow();
        logic [1:0] tag, resp;
        logic [0:31] op1, op2, data;
        logic [0:3] status;

        $display("---------- STARTING TEST: operations_on_req1_without_overflow ----------");
        status = 0;
        initialize();
        deassert_reset();
        repeat(2)@(posedge c_clk);

        for(int i=0; i<4;i++) begin
            tag = i;
            op1 = $urandom_range(5,10);
            op2 = $urandom_range(0,5);
            if(i==0) begin
                fork
                    driver_req1(`ADD,op1,op2,tag);
                    monitor_req1(tag, resp, data);
                join
                scoreboard(`ADD,op1,op2,resp,data,status[0]);
            end
            else if(i==1) begin
                fork
                    driver_req1(`SUBTRACT,op1,op2,tag);
                    monitor_req1(tag, resp, data);
                join
                scoreboard(`SUBTRACT,op1,op2,resp,data,status[1]);
            end
            else if(i==2) begin
                fork
                    driver_req1(`SRL,op1,op2,tag);
                    monitor_req1(tag, resp, data);
                join
                scoreboard(`SRL,op1,op2,resp,data,status[2]);
            end
            else if(i==3) begin
                fork
                    driver_req1(`SLL,op1,op2,tag);
                    monitor_req1(tag, resp, data);
                join
                scoreboard(`SLL,op1,op2,resp,data,status[3]);
            end
        end

        if(status == 4'b0)
            $display("TEST PASSED - 'operations_on_req1_without_overflow'");
        else
            $display("TEST FAILED - 'operations_on_req1_without_overflow'");
    
        $display("---------- TEST END: operations_on_req1_without_overflow ----------");
    endtask

    task operations_on_req2_without_overflow();
        logic [1:0] tag, resp;
        logic [0:31] op1, op2, data;
        logic [0:3] status;

        $display("---------- STARTING TEST: operations_on_req2_without_overflow ----------");
        status = 0;
        initialize();
        deassert_reset();
        repeat(2)@(posedge c_clk);

        for(int i=0; i<4;i++) begin
            tag = i;
            op1 = $urandom_range(5,10);
            op2 = $urandom_range(0,5);
            if(i==0) begin
                fork
                    driver_req2(`ADD,op1,op2,tag);
                    monitor_req2(tag, resp, data);
                join
                scoreboard(`ADD,op1,op2,resp,data,status[0]);
            end
            else if(i==1) begin
                fork
                    driver_req2(`SUBTRACT,op1,op2,tag);
                    monitor_req2(tag, resp, data);
                join
                scoreboard(`SUBTRACT,op1,op2,resp,data,status[1]);
            end
            else if(i==2) begin
                fork
                    driver_req2(`SRL,op1,op2,tag);
                    monitor_req2(tag, resp, data);
                join
                scoreboard(`SRL,op1,op2,resp,data,status[2]);
            end
            else if(i==3) begin
                fork
                    driver_req2(`SLL,op1,op2,tag);
                    monitor_req2(tag, resp, data);
                join
                scoreboard(`SLL,op1,op2,resp,data,status[3]);
            end
        end

        if(status == 4'b0)
            $display("TEST PASSED - 'operations_on_req2_without_overflow'");
        else
            $display("TEST FAILED - 'operations_on_req2_without_overflow'");
    
        $display("---------- TEST END: operations_on_req2_without_overflow ----------");
    endtask

    task operations_on_req3_without_overflow();
        logic [1:0] tag, resp;
        logic [0:31] op1, op2, data;
        logic [0:3] status;

        $display("---------- STARTING TEST: operations_on_req3_without_overflow ----------");
        status = 0;
        initialize();
        deassert_reset();
        repeat(2)@(posedge c_clk);

        for(int i=0; i<4;i++) begin
            tag = i;
            op1 = $urandom_range(5,10);
            op2 = $urandom_range(0,5);
            if(i==0) begin
                fork
                    driver_req3(`ADD,op1,op2,tag);
                    monitor_req3(tag, resp, data);
                join
                scoreboard(`ADD,op1,op2,resp,data,status[0]);
            end
            else if(i==1) begin
                fork
                    driver_req3(`SUBTRACT,op1,op2,tag);
                    monitor_req3(tag, resp, data);
                join
                scoreboard(`SUBTRACT,op1,op2,resp,data,status[1]);
            end
            else if(i==2) begin
                fork
                    driver_req3(`SRL,op1,op2,tag);
                    monitor_req3(tag, resp, data);
                join
                scoreboard(`SRL,op1,op2,resp,data,status[2]);
            end
            else if(i==3) begin
                fork
                    driver_req3(`SLL,op1,op2,tag);
                    monitor_req3(tag, resp, data);
                join
                scoreboard(`SLL,op1,op2,resp,data,status[3]);
            end
        end

        if(status == 4'b0)
            $display("TEST PASSED - 'operations_on_req3_without_overflow'");
        else
            $display("TEST FAILED - 'operations_on_req3_without_overflow'");
    
        $display("---------- TEST END: operations_on_req3_without_overflow ----------");
    endtask

    task operations_on_req4_without_overflow();
        logic [1:0] tag, resp;
        logic [0:31] op1, op2, data;
        logic [0:3] status;

        $display("---------- STARTING TEST: operations_on_req4_without_overflow ----------");
        status = 0;
        initialize();
        deassert_reset();
        repeat(2)@(posedge c_clk);

        for(int i=0; i<4;i++) begin
            tag = i;
            op1 = $urandom_range(5,10);
            op2 = $urandom_range(0,5);
            if(i==0) begin
                fork
                    driver_req4(`ADD,op1,op2,tag);
                    monitor_req4(tag, resp, data);
                join
                scoreboard(`ADD,op1,op2,resp,data,status[0]);
            end
            else if(i==1) begin
                fork
                    driver_req4(`SUBTRACT,op1,op2,tag);
                    monitor_req4(tag, resp, data);
                join
                scoreboard(`SUBTRACT,op1,op2,resp,data,status[1]);
            end
            else if(i==2) begin
                fork
                    driver_req4(`SRL,op1,op2,tag);
                    monitor_req4(tag, resp, data);
                join
                scoreboard(`SRL,op1,op2,resp,data,status[2]);
            end
            else if(i==3) begin
                fork
                    driver_req4(`SLL,op1,op2,tag);
                    monitor_req4(tag, resp, data);
                join
                scoreboard(`SLL,op1,op2,resp,data,status[3]);
            end
        end

        if(status == 4'b0)
            $display("TEST PASSED - 'operations_on_req4_without_overflow'");
        else
            $display("TEST FAILED - 'operations_on_req4_without_overflow'");
    
        $display("---------- TEST END: operations_on_req4_without_overflow ----------");
    endtask

    task sanity_test_for_add_on_req1();
        logic [1:0] tag, resp;
        logic [0:31] op1, op2, data;
        logic [0:7] status;

        $display("---------- STARTING TEST: sanity_test_for_add_on_req1 ----------");
        status = 0;
        initialize();
        deassert_reset();
        repeat(2)@(posedge c_clk);

        for(int i = 0;i < 8; i++) begin
            if(i == 0) begin
                tag = i;
                op1 = 0;
                op2 = 0;
            end else if(i == 1) begin
                tag = i;
                op1 = 32'h77777777;
                op2 = 32'h88888888;
            end else begin
                tag = i;
                op1 = $urandom();
                op2 = $urandom();
            end
            
            fork
                driver_req1(`ADD,op1,op2,tag);
                monitor_req1(tag, resp, data);
            join
                scoreboard(`ADD,op1,op2,resp,data,status[i]);
        end

        if(status == 8'b0)
            $display("TEST PASSED - 'sanity_test_for_add_on_req1'");
        else
            $display("TEST FAILED - 'sanity_test_for_add_on_req1'");
    
        $display("---------- TEST END: sanity_test_for_add_on_req1 ----------");
    endtask

    task sanity_test_for_subtract_on_req1();
        logic [1:0] tag, resp;
        logic [0:31] op1, op2, data;
        logic [0:7] status;

        $display("---------- STARTING TEST: sanity_test_for_subtract_on_req1 ----------");
        status = 0;
        initialize();
        deassert_reset();
        repeat(2)@(posedge c_clk);

        for(int i = 0;i < 8; i++) begin
            if(i == 0) begin
                tag = i;
                op1 = 0;
                op2 = 0;
            end else if(i == 1) begin
                tag = i;
                op1 = 32'h88888888;
                op2 = 32'h66666666;
            end else if(i == 2) begin
                tag = i;
                op1 = 32'h88888888;
                op2 = 32'hAAAAAAAA;
            end else begin
                tag = i;
                op1 = $urandom();
                op2 = $urandom();
            end
            
            fork
                driver_req1(`SUBTRACT,op1,op2,tag);
                monitor_req1(tag, resp, data);
            join
                scoreboard(`SUBTRACT,op1,op2,resp,data,status[i]);
        end

        if(status == 8'b0)
            $display("TEST PASSED - 'sanity_test_for_subtract_on_req1'");
        else
            $display("TEST FAILED - 'sanity_test_for_subtract_on_req1'");
    
        $display("---------- TEST END: sanity_test_for_subtract_on_req1 ----------");
    endtask

    task sanity_test_for_sll_on_req1();
        logic [1:0] tag, resp;
        logic [0:31] op1, op2, data;
        logic [0:34] status;

        $display("---------- STARTING TEST: sanity_test_for_sll_on_req1 ----------");
        status = 0;
        initialize();
        deassert_reset();
        repeat(2)@(posedge c_clk);

        for(int i = 0;i < 35; i++) begin
            if(i < 32) begin
                tag = i;
                op1 = 1;
                op2 = i;
            end else begin
                tag = i;
                op1 = $urandom();
                op2 = $urandom();
            end
            
            fork
                driver_req1(`SLL,op1,op2,tag);
                monitor_req1(tag, resp, data);
            join
                scoreboard(`SLL,op1,op2,resp,data,status[i]);
        end

        if(status == 35'b0)
            $display("TEST PASSED - 'sanity_test_for_sll_on_req1'");
        else
            $display("TEST FAILED - 'sanity_test_for_sll_on_req1'");
    
        $display("---------- TEST END: sanity_test_for_sll_on_req1 ----------");
    endtask

    task sanity_test_for_srl_on_req1();
        logic [1:0] tag, resp;
        logic [0:31] op1, op2, data;
        logic [0:34] status;

        $display("---------- STARTING TEST: sanity_test_for_srl_on_req1 ----------");
        status = 0;
        initialize();
        deassert_reset();
        repeat(2)@(posedge c_clk);

        for(int i = 0;i < 35; i++) begin
            if(i < 32) begin
                tag = i;
                op1 = 32'h80000000;
                op2 = i;
            end else begin
                tag = i;
                op1 = $urandom();
                op2 = $urandom();
            end
            
            fork
                driver_req1(`SRL,op1,op2,tag);
                monitor_req1(tag, resp, data);
            join
                scoreboard(`SRL,op1,op2,resp,data,status[i]);
        end

        if(status == 35'b0)
            $display("TEST PASSED - 'sanity_test_for_srl_on_req1'");
        else
            $display("TEST FAILED - 'sanity_test_for_srl_on_req1'");
    
        $display("---------- TEST END: sanity_test_for_srl_on_req1 ----------");
    endtask

    task sanity_test_for_add_on_req2();
        logic [1:0] tag, resp;
        logic [0:31] op1, op2, data;
        logic [0:7] status;

        $display("---------- STARTING TEST: sanity_test_for_add_on_req2 ----------");
        status = 0;
        initialize();
        deassert_reset();
        repeat(2)@(posedge c_clk);

        for(int i = 0;i < 8; i++) begin
            if(i == 0) begin
                tag = i;
                op1 = 0;
                op2 = 0;
            end else if(i == 1) begin
                tag = i;
                op1 = 32'h77777777;
                op2 = 32'h88888888;
            end else begin
                tag = i;
                op1 = $urandom();
                op2 = $urandom();
            end
            
            fork
                driver_req2(`ADD,op1,op2,tag);
                monitor_req2(tag, resp, data);
            join
                scoreboard(`ADD,op1,op2,resp,data,status[i]);
        end

        if(status == 8'b0)
            $display("TEST PASSED - 'sanity_test_for_add_on_req2'");
        else
            $display("TEST FAILED - 'sanity_test_for_add_on_req2'");
    
        $display("---------- TEST END: sanity_test_for_add_on_req2 ----------");
    endtask

    task sanity_test_for_subtract_on_req2();
        logic [1:0] tag, resp;
        logic [0:31] op1, op2, data;
        logic [0:7] status;

        $display("---------- STARTING TEST: sanity_test_for_subtract_on_req2 ----------");
        status = 0;
        initialize();
        deassert_reset();
        repeat(2)@(posedge c_clk);

        for(int i = 0;i < 8; i++) begin
            if(i == 0) begin
                tag = i;
                op1 = 0;
                op2 = 0;
            end else if(i == 1) begin
                tag = i;
                op1 = 32'h88888888;
                op2 = 32'h66666666;
            end else if(i == 2) begin
                tag = i;
                op1 = 32'h88888888;
                op2 = 32'hAAAAAAAA;
            end else begin
                tag = i;
                op1 = $urandom();
                op2 = $urandom();
            end
            
            fork
                driver_req2(`SUBTRACT,op1,op2,tag);
                monitor_req2(tag, resp, data);
            join
                scoreboard(`SUBTRACT,op1,op2,resp,data,status[i]);
        end

        if(status == 8'b0)
            $display("TEST PASSED - 'sanity_test_for_subtract_on_req2'");
        else
            $display("TEST FAILED - 'sanity_test_for_subtract_on_req2'");
    
        $display("---------- TEST END: sanity_test_for_subtract_on_req2 ----------");
    endtask

    task sanity_test_for_sll_on_req2();
        logic [1:0] tag, resp;
        logic [0:31] op1, op2, data;
        logic [0:34] status;

        $display("---------- STARTING TEST: sanity_test_for_sll_on_req2 ----------");
        status = 0;
        initialize();
        deassert_reset();
        repeat(2)@(posedge c_clk);

        for(int i = 0;i < 35; i++) begin
            if(i < 32) begin
                tag = i;
                op1 = 1;
                op2 = i;
            end else begin
                tag = i;
                op1 = $urandom();
                op2 = $urandom();
            end
            
            fork
                driver_req2(`SLL,op1,op2,tag);
                monitor_req2(tag, resp, data);
            join
                scoreboard(`SLL,op1,op2,resp,data,status[i]);
        end

        if(status == 35'b0)
            $display("TEST PASSED - 'sanity_test_for_sll_on_req2'");
        else
            $display("TEST FAILED - 'sanity_test_for_sll_on_req2'");
    
        $display("---------- TEST END: sanity_test_for_sll_on_req2 ----------");
    endtask

    task sanity_test_for_srl_on_req2();
        logic [1:0] tag, resp;
        logic [0:31] op1, op2, data;
        logic [0:34] status;

        $display("---------- STARTING TEST: sanity_test_for_srl_on_req2 ----------");
        status = 0;
        initialize();
        deassert_reset();
        repeat(2)@(posedge c_clk);

        for(int i = 0;i < 35; i++) begin
            if(i < 32) begin
                tag = i;
                op1 = 32'h80000000;
                op2 = i;
            end else begin
                tag = i;
                op1 = $urandom();
                op2 = $urandom();
            end
            
            fork
                driver_req2(`SRL,op1,op2,tag);
                monitor_req2(tag, resp, data);
            join
                scoreboard(`SRL,op1,op2,resp,data,status[i]);
        end

        if(status == 35'b0)
            $display("TEST PASSED - 'sanity_test_for_srl_on_req2'");
        else
            $display("TEST FAILED - 'sanity_test_for_srl_on_req2'");
    
        $display("---------- TEST END: sanity_test_for_srl_on_req2 ----------");
    endtask

    task sanity_test_for_add_on_req3();
        logic [1:0] tag, resp;
        logic [0:31] op1, op2, data;
        logic [0:7] status;

        $display("---------- STARTING TEST: sanity_test_for_add_on_req3 ----------");
        status = 0;
        initialize();
        deassert_reset();
        repeat(2)@(posedge c_clk);

        for(int i = 0;i < 8; i++) begin
            if(i == 0) begin
                tag = i;
                op1 = 0;
                op2 = 0;
            end else if(i == 1) begin
                tag = i;
                op1 = 32'h77777777;
                op2 = 32'h88888888;
            end else begin
                tag = i;
                op1 = $urandom();
                op2 = $urandom();
            end
            
            fork
                driver_req3(`ADD,op1,op2,tag);
                monitor_req3(tag, resp, data);
            join
                scoreboard(`ADD,op1,op2,resp,data,status[i]);
        end

        if(status == 8'b0)
            $display("TEST PASSED - 'sanity_test_for_add_on_req3'");
        else
            $display("TEST FAILED - 'sanity_test_for_add_on_req3'");
    
        $display("---------- TEST END: sanity_test_for_add_on_req3 ----------");
    endtask

    task sanity_test_for_subtract_on_req3();
        logic [1:0] tag, resp;
        logic [0:31] op1, op2, data;
        logic [0:7] status;

        $display("---------- STARTING TEST: sanity_test_for_subtract_on_req3 ----------");
        status = 0;
        initialize();
        deassert_reset();
        repeat(2)@(posedge c_clk);

        for(int i = 0;i < 8; i++) begin
            if(i == 0) begin
                tag = i;
                op1 = 0;
                op2 = 0;
            end else if(i == 1) begin
                tag = i;
                op1 = 32'h88888888;
                op2 = 32'h66666666;
            end else if(i == 2) begin
                tag = i;
                op1 = 32'h88888888;
                op2 = 32'hAAAAAAAA;
            end else begin
                tag = i;
                op1 = $urandom();
                op2 = $urandom();
            end
            
            fork
                driver_req3(`SUBTRACT,op1,op2,tag);
                monitor_req3(tag, resp, data);
            join
                scoreboard(`SUBTRACT,op1,op2,resp,data,status[i]);
        end

        if(status == 8'b0)
            $display("TEST PASSED - 'sanity_test_for_subtract_on_req3'");
        else
            $display("TEST FAILED - 'sanity_test_for_subtract_on_req3'");
    
        $display("---------- TEST END: sanity_test_for_subtract_on_req3 ----------");
    endtask

    task sanity_test_for_sll_on_req3();
        logic [1:0] tag, resp;
        logic [0:31] op1, op2, data;
        logic [0:34] status;

        $display("---------- STARTING TEST: sanity_test_for_sll_on_req3 ----------");
        status = 0;
        initialize();
        deassert_reset();
        repeat(2)@(posedge c_clk);

        for(int i = 0;i < 35; i++) begin
            if(i < 32) begin
                tag = i;
                op1 = 1;
                op2 = i;
            end else begin
                tag = i;
                op1 = $urandom();
                op2 = $urandom();
            end
            
            fork
                driver_req3(`SLL,op1,op2,tag);
                monitor_req3(tag, resp, data);
            join
                scoreboard(`SLL,op1,op2,resp,data,status[i]);
        end

        if(status == 35'b0)
            $display("TEST PASSED - 'sanity_test_for_sll_on_req3'");
        else
            $display("TEST FAILED - 'sanity_test_for_sll_on_req3'");
    
        $display("---------- TEST END: sanity_test_for_sll_on_req3 ----------");
    endtask

    task sanity_test_for_srl_on_req3();
        logic [1:0] tag, resp;
        logic [0:31] op1, op2, data;
        logic [0:34] status;

        $display("---------- STARTING TEST: sanity_test_for_srl_on_req3 ----------");
        status = 0;
        initialize();
        deassert_reset();
        repeat(2)@(posedge c_clk);

        for(int i = 0;i < 35; i++) begin
            if(i < 32) begin
                tag = i;
                op1 = 32'h80000000;
                op2 = i;
            end else begin
                tag = i;
                op1 = $urandom();
                op2 = $urandom();
            end
            
            fork
                driver_req3(`SRL,op1,op2,tag);
                monitor_req3(tag, resp, data);
            join
                scoreboard(`SRL,op1,op2,resp,data,status[i]);
        end

        if(status == 35'b0)
            $display("TEST PASSED - 'sanity_test_for_srl_on_req3'");
        else
            $display("TEST FAILED - 'sanity_test_for_srl_on_req3'");
    
        $display("---------- TEST END: sanity_test_for_srl_on_req3 ----------");
    endtask

    task sanity_test_for_add_on_req4();
        logic [1:0] tag, resp;
        logic [0:31] op1, op2, data;
        logic [0:7] status;

        $display("---------- STARTING TEST: sanity_test_for_add_on_req4 ----------");
        status = 0;
        initialize();
        deassert_reset();
        repeat(2)@(posedge c_clk);

        for(int i = 0;i < 8; i++) begin
            if(i == 0) begin
                tag = i;
                op1 = 0;
                op2 = 0;
            end else if(i == 1) begin
                tag = i;
                op1 = 32'h77777777;
                op2 = 32'h88888888;
            end else begin
                tag = i;
                op1 = $urandom();
                op2 = $urandom();
            end
            
            fork
                driver_req4(`ADD,op1,op2,tag);
                monitor_req4(tag, resp, data);
            join
                scoreboard(`ADD,op1,op2,resp,data,status[i]);
        end

        if(status == 8'b0)
            $display("TEST PASSED - 'sanity_test_for_add_on_req4'");
        else
            $display("TEST FAILED - 'sanity_test_for_add_on_req4'");
    
        $display("---------- TEST END: sanity_test_for_add_on_req4 ----------");
    endtask

    task sanity_test_for_subtract_on_req4();
        logic [1:0] tag, resp;
        logic [0:31] op1, op2, data;
        logic [0:7] status;

        $display("---------- STARTING TEST: sanity_test_for_subtract_on_req4 ----------");
        status = 0;
        initialize();
        deassert_reset();
        repeat(2)@(posedge c_clk);

        for(int i = 0;i < 8; i++) begin
            if(i == 0) begin
                tag = i;
                op1 = 0;
                op2 = 0;
            end else if(i == 1) begin
                tag = i;
                op1 = 32'h88888888;
                op2 = 32'h66666666;
            end else if(i == 2) begin
                tag = i;
                op1 = 32'h88888888;
                op2 = 32'hAAAAAAAA;
            end else begin
                tag = i;
                op1 = $urandom();
                op2 = $urandom();
            end
            
            fork
                driver_req4(`SUBTRACT,op1,op2,tag);
                monitor_req4(tag, resp, data);
            join
                scoreboard(`SUBTRACT,op1,op2,resp,data,status[i]);
        end

        if(status == 8'b0)
            $display("TEST PASSED - 'sanity_test_for_subtract_on_req4'");
        else
            $display("TEST FAILED - 'sanity_test_for_subtract_on_req4'");
    
        $display("---------- TEST END: sanity_test_for_subtract_on_req4 ----------");
    endtask

    task sanity_test_for_sll_on_req4();
        logic [1:0] tag, resp;
        logic [0:31] op1, op2, data;
        logic [0:34] status;

        $display("---------- STARTING TEST: sanity_test_for_sll_on_req4 ----------");
        status = 0;
        initialize();
        deassert_reset();
        repeat(2)@(posedge c_clk);

        for(int i = 0;i < 35; i++) begin
            if(i < 32) begin
                tag = i;
                op1 = 1;
                op2 = i;
            end else begin
                tag = i;
                op1 = $urandom();
                op2 = $urandom();
            end
            
            fork
                driver_req4(`SLL,op1,op2,tag);
                monitor_req4(tag, resp, data);
            join
                scoreboard(`SLL,op1,op2,resp,data,status[i]);
        end

        if(status == 35'b0)
            $display("TEST PASSED - 'sanity_test_for_sll_on_req4'");
        else
            $display("TEST FAILED - 'sanity_test_for_sll_on_req4'");
    
        $display("---------- TEST END: sanity_test_for_sll_on_req4 ----------");
    endtask

    task sanity_test_for_srl_on_req4();
        logic [1:0] tag, resp;
        logic [0:31] op1, op2, data;
        logic [0:34] status;

        $display("---------- STARTING TEST: sanity_test_for_srl_on_req4 ----------");
        status = 0;
        initialize();
        deassert_reset();
        repeat(2)@(posedge c_clk);

        for(int i = 0;i < 35; i++) begin
            if(i < 32) begin
                tag = i;
                op1 = 32'h80000000;
                op2 = i;
            end else begin
                tag = i;
                op1 = $urandom();
                op2 = $urandom();
            end
            
            fork
                driver_req4(`SRL,op1,op2,tag);
                monitor_req4(tag, resp, data);
            join
                scoreboard(`SRL,op1,op2,resp,data,status[i]);
        end

        if(status == 35'b0)
            $display("TEST PASSED - 'sanity_test_for_srl_on_req4'");
        else
            $display("TEST FAILED - 'sanity_test_for_srl_on_req4'");
    
        $display("---------- TEST END: sanity_test_for_srl_on_req4 ----------");
    endtask

    task simultaneous_operation_on_all_channels();
        $display("---------- TEST START: simultaneous_operation_on_all_channels ----------");
        fork
            begin
                sanity_test_for_add_on_req1();
                sanity_test_for_subtract_on_req1();
                sanity_test_for_sll_on_req1();
                sanity_test_for_srl_on_req1();
            end
            begin
                sanity_test_for_add_on_req2();
                sanity_test_for_subtract_on_req2();
                sanity_test_for_sll_on_req2();
                sanity_test_for_srl_on_req2();
            end
            begin
                sanity_test_for_add_on_req3();
                sanity_test_for_subtract_on_req3();
                sanity_test_for_sll_on_req3();
                sanity_test_for_srl_on_req3();
            end
            begin
                sanity_test_for_add_on_req4();
                sanity_test_for_subtract_on_req4();
                sanity_test_for_sll_on_req4();
                sanity_test_for_srl_on_req4();
            end
        join
        $display("---------- TEST END: simultaneous_operation_on_all_channels ----------");
    endtask

    initial begin
        operation_with_reset_HIGH();
        operations_on_req1_without_overflow();
        operations_on_req2_without_overflow();
        operations_on_req3_without_overflow();
        operations_on_req4_without_overflow();
        sanity_test_for_add_on_req1();
        sanity_test_for_subtract_on_req1();
        sanity_test_for_sll_on_req1();
        sanity_test_for_srl_on_req1();
        sanity_test_for_add_on_req2();
        sanity_test_for_subtract_on_req2();
        sanity_test_for_sll_on_req2();
        sanity_test_for_srl_on_req2();
        sanity_test_for_add_on_req3();
        sanity_test_for_subtract_on_req3();
        sanity_test_for_sll_on_req3();
        sanity_test_for_srl_on_req3();
        sanity_test_for_add_on_req4();
        sanity_test_for_subtract_on_req4();
        sanity_test_for_sll_on_req4();
        sanity_test_for_srl_on_req4();
        //simultaneous_operation_on_all_channels();

        repeat(10)@(posedge c_clk);
        $stop;
    end

endmodule

