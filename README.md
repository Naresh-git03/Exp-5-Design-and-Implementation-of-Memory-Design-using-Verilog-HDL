# Exp-5-Design-and-Simulate the-Memory-Design-using-Verilog-HDL
#Aim
To design and simulate a RAM,ROM,FIFO using Verilog HDL, and verify its functionality through a testbench in the Vivado 2023.1 environment.
Apparatus Required
Vivado 2023.1
Procedure
1. Launch Vivado 2023.1
Open Vivado and create a new project.
2. Design the Verilog Code
Write the Verilog code for the RAM,ROM,FIFO
3. Create the Testbench
Write a testbench to simulate the memory behavior. The testbench should apply various and monitor the corresponding output.
4. Create the Verilog Files
Create both the design module and the testbench in the Vivado project.
5. Run Simulation
Run the behavioral simulation to verify the output.
6. Observe the Waveforms
Analyze the output waveforms in the simulation window, and verify that the correct read and write operation.
7. Save and Document Results
Capture screenshots of the waveform and save the simulation logs. These will be included in the lab report.

# Code
# RAM
// Verilog code
```
module ram(input clk,input rst,input en,input [7:0] datain,input [9:0] address,output reg [7:0] dataout);
    reg [7:0] mem [1023:0];
    always @(posedge clk) begin
    if (rst) begin
    dataout <= 8'b0;
    end
    else if (en) begin
    mem[address] <= datain;
    end
    else begin
    dataout <= mem[address];
    end
end
endmodule
```

// Test bench
```
module ram_tb;
reg clk_t, rst_t, en_t;
reg [7:0] datain_t;
reg [9:0] address_t;
  wire [7:0] dataout_t;
  ram dut (.clk(clk_t),.rst(rst_t) );
  initial begin
    clk_t = 1'b0;
    rst_t = 1'b1;
    #100
    rst_t = 1'b0;
    en_t=1'b1;
    address_t = 10'd800;
    datain_t = 8'd50;
    #100;
    address_t = 10'd950;
    datain_t = 8'd60;
    #100;
    en_t = 1'b0;
    address_t = 10'd800;
    #100;
    address_t = 10'd900;
    #100;
  end
    always #10 clk_t=~clk_t;
    endmodule
```

// output Waveform
<img width="1920" height="1200" alt="Screenshot (26)" src="https://github.com/user-attachments/assets/70eff8e2-aaa3-434b-9b68-9d3b3704f6a9" />

# ROM
 // write verilog code for ROM using $random
 ```
module romm(input clk,rst,
    input [9:0]address,
    output reg [7:0]dataout
  );
    reg [7:0] mem_1kb_rom[1023:0];
    integer i;
    initial
        begin
            for(i=0;i<1024;i=i+1)
                mem_1kb_rom[i] = $random;
            end
     always@(posedge clk)
        begin
            if(rst)
                dataout <= 8'b0;
            else
                dataout <= mem_1kb_rom[address];
          end
endmodule
```
 // Test bench
 ```
module romm_tb;
    reg clk_t,rst_t;
    reg [9:0]address_t;
    wire [7:0]dataout_t;
    
    romm dut(.clk(clk_t),.rst(rst_t),.address(address_t),.dataout(dataout_t));
    
    initial
       begin
           clk_t = 1'b0;
           rst_t = 1'b1;
        #100
           rst_t = 1'b0;
           address_t = 10'd700;
        #100
           address_t = 10'd800;
        #100
           address_t = 10'd900;
       end
    always
        #10 clk_t = ~clk_t;   
endmodule
```

// output Waveform
<img width="1920" height="1200" alt="Screenshot (27)" src="https://github.com/user-attachments/assets/5e381f5a-9913-4b19-9bf9-a461a07152a3" />

 # FIFO
 // write verilog code for FIFO
 ```
    module fifo_mem #(
    parameter DATA_WIDTH = 8,
    parameter DEPTH = 16
)(
    input clk, rst,
    input wr_en, rd_en,
    input [DATA_WIDTH-1:0] din,
    output reg [DATA_WIDTH-1:0] dout,
    output reg full, empty
);
    reg [DATA_WIDTH-1:0] fifo_mem [0:DEPTH-1];
    reg [$clog2(DEPTH):0] wr_ptr;
    reg [$clog2(DEPTH):0] rd_ptr;
    reg [$clog2(DEPTH):0] count;
    integer i;
    always @(posedge clk) begin
        if (rst) begin
            wr_ptr <= 0;
            rd_ptr <= 0;
            count  <= 0;
            dout   <= 0;
            full   <= 0;
            empty  <= 1;
            for (i=0; i<DEPTH; i=i+1)
                fifo_mem[i] <= 0;
        end else begin
            if (wr_en && !full) begin
                fifo_mem[wr_ptr] <= din;
                wr_ptr <= wr_ptr + 1;
                count <= count + 1;
            end
            if (rd_en && !empty) begin
                dout <= fifo_mem[rd_ptr];
                rd_ptr <= rd_ptr + 1;
                count <= count - 1;
            end
            full  <= (count == DEPTH-1);
            empty <= (count == 0);
        end
    end
endmodule
```

 // Test bench
 ```
module tb_fifo;
    parameter DATA_WIDTH = 8;
    parameter DEPTH = 16;
    reg clk, rst;
    reg wr_en, rd_en;
    reg [DATA_WIDTH-1:0] din;
    wire [DATA_WIDTH-1:0] dout;
    wire full, empty;
    fifo_mem #(
        .DATA_WIDTH(DATA_WIDTH),
        .DEPTH(DEPTH)
    ) uut (
        .clk(clk),
        .rst(rst),
        .wr_en(wr_en),
        .rd_en(rd_en),
        .din(din),
        .dout(dout),
        .full(full),
        .empty(empty)
    );
    always #5 clk = ~clk;
    initial begin
        clk = 0; rst = 1; wr_en = 0; rd_en = 0; din = 0;
        #20 rst = 0;
        repeat (5) begin
            @(posedge clk);
            wr_en = 1;
            din = $random % 256;
        end
        @(posedge clk) wr_en = 0;
        repeat (5) begin
            @(posedge clk);
            rd_en = 1;
        end
        @(posedge clk) rd_en = 0;
        #50 $stop;
    end
endmodule
```

// output Waveform

<img width="1920" height="1200" alt="Screenshot (28)" src="https://github.com/user-attachments/assets/249b22b2-1b16-4cd9-bd63-140e310dc5ce" />


# Conclusion
The RAM, ROM, FIFO memory with read and write operations was designed and successfully simulated using Verilog HDL. The testbench verified both the write and read functionalities by simulating the memory operations and observing the output waveforms. The experiment demonstrates how to implement memory operations in Verilog, effectively modeling both the reading and writing processes.
 
 

