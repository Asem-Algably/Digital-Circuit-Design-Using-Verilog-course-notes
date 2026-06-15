a modulus counter is a type of counters that counts till a specific value and then resets

### hardcoded modulus counter
the reason this counter is called hardcoded is that the modulus value is hardcoded into the verilog description instead of being inputted dynamically
```verilog
module mod_counter_hardcoded
    #(parameter BTS = 4)(
    input clk,
    input reset_n,
    input enable,
    output [BTS-1: 0] Q
    );
    
    reg [BTS - 1: 0] Q_next, Q_reg;
    wire done;
    
    always @(posedge clk, negedge reset_n)
    begin
        if(!reset_n)
            Q_reg <= 1'b0;
        else if (enable)
            Q_reg <= Q_next;
        else
            Q_reg <= Q_reg;
    end
    
    assign done = (Q_reg == 7)? 1'b1: 1'b0; 
    
    // next state logic
    always @(Q_reg)
    begin
        if(done)
            Q_next = 1'b0;
        else
            Q_next = Q_reg + 1;
    end
    
    // output logic
    assign Q = Q_reg;
 
endmodule
```

that was a modulus 7 counter and here is how to test it
```verilog
module mod_counter_hardcoded_tb(
    );
    //1) Declare local reg and wire identifiers
    localparam BITS = 4;
    reg clk, reset_n, enable;
    wire [3:0] Q;
    
    //2) Instantiate the module under test
    mod_counter_hardcoded #(.BITS(BITS))uut (
        .clk(clk),
        .reset_n(reset_n),
        .enable(enable),
        .Q(Q)
        );
        
    //3) Specify a stopwatch to stop the simulation
    initial
    begin 
        #200 $stop;
    end
    
    //4) Generate stimuli, using initial and always
    // clock cycle
    localparam T = 10;
    always
    begin
        clk = 1'b0;
        #(T / 2); //<= you have to put T/2 inside parentheses
                  // in order for the math expression to be evaluated first 
        clk = 1'b1;
        #(T / 2);
    end
    
    initial
    begin
        reset_n = 1'b0;
        enable = 1'b0;
        #2 reset_n = 1'b1;
        #2 enable = 1'b1;
    end
    
endmodule
```

### a dynamic modulus counter
now let’s code a one that loads the modulus value from an input bus
```verilog
module mod_counter_input
    #(parameter BTS = 4)(
    input clk,
    input reset_n,
    input enable,
    input [BTS-1: 0] FINAL_VALUE,
    output [BTS-1: 0] Q
    );
    
    reg [BTS - 1: 0] Q_next, Q_reg;
    wire done;
    
    always @(posedge clk, negedge reset_n)
    begin
        if(!reset_n)
            Q_reg <= 1'b0;
        else if (enable)
            Q_reg <= Q_next;
        else
            Q_reg <= Q_reg;
    end
    
    assign done = (Q_reg == FINAL_VALUE); 
    
    // next state logic
    always @(Q_reg)
    begin
        if(done)
            Q_next = 1'b0;
        else
            Q_next = Q_reg + 1;
    end
    
    // output logic
    assign Q = Q_reg;
 
endmodule
```

but actually most of the time we don’t need to change the value of the modulus on the fly so we can instead provide the modulus value via a generalized parameter
```verilog
module mod_counter_parameter
    #(parameter FINAL_VALUE = 9)(
    input clk,
    input reset_n,
    input enable,
    output [BTS-1: 0] Q
    );
    
    localparam BTS = $clog2(FINAL_VALUE);
    reg [BTS - 1: 0] Q_next, Q_reg;
    wire done;
    
    always @(posedge clk, negedge reset_n)
    begin
        if(!reset_n)
            Q_reg <= 1'b0;
        else if (enable)
            Q_reg <= Q_next;
        else
            Q_reg <= Q_reg;
    end
    
    assign done = (Q_reg == FINAL_VALUE); 
    
    // next state logic
    always @(Q_reg)
    begin
        if(done)
            Q_next = 1'b0;
        else
            Q_next = Q_reg + 1;
    end
    
    // output logic
    assign Q = Q_reg;
 
endmodule
```

notice the use of the local parameter here, the goal is to obtain the value of the number of bits required to count up to this modulus, note that we here used local parameters instead of registers or wires because we don’t want a physical component here we just want a value place holder like the `define` macro in C but on a local scope

>[!TIP]
>Speaking of text macros, Verilog actually provides full preprocessor support to the  `define` directive, which behaves very similarly to C. While a `localparam` is strictly a scoped module constant, a `define` macro acts as a global text replacement tool. It even includes C-style features, including multiline definitions using the trailing backslash (`\`) and functional macros that accept text arguments, their syntax is a teeny tiny bit different from C but mostly they are the same

>[!NOTE]
>Also, notice that we can use the local parameter `BTS` before it is defined. This is different from sequential programming languages because, as we discussed before, Verilog does not execute code line by line. Instead, the code is interpreted concurrently, allowing declarations to be referenced regardless of their order in the module.

