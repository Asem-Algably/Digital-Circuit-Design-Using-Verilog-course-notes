most of the time when we work with a timer in a digital system there is a good chance it’s built of a timer, in this module we will create a timer using a modulus counter

### timer parameter
this is so similar to modulus counter with the modulus being the maximum time we want to count to, also a clear different here between it and the counters is that we don’t care about the count value but only when it finishes counting (the done signal), note that in some systems you might need the count signal

```verilog
module timer_parameter
    #(parameter FINAL_VALUE = 255)(
        input clk,
        input reset_n,
        input enable,
        output done
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

we calculate the maximum count from the frequency of the clock (in other words the periodic time) and the maximum time we want to count to

by putting the timer done signal into a T flip flop we can create a clock generator

we can also pass the final timer value to the timer via an input bus instead of a parameter which allow us to change the value of the timer on the fly
```verilog
module timer_input
    #(parameter BITS = 4)(
    input clk,
    input reset_n,
    input enable,
    input [BITS - 1:0] FINAL_VALUE,
//    output [BITS - 1:0] Q,
    output done
    );
    
    reg [BITS - 1:0] Q_reg, Q_next;
    
    always @(posedge clk, negedge reset_n)
    begin
        if (~reset_n)
            Q_reg <= 'b0;
        else if(enable)
            Q_reg <= Q_next;
        else
            Q_reg <= Q_reg;
    end
    
    // Next state logic
    assign done = Q_reg == FINAL_VALUE;

    always @(*)
        Q_next = done? 'b0: Q_reg + 1;
    
    
endmodule
```

we can demonstrate the change of fly concept using our testbench output but first let’s set our testbench to change the value of the counter while  working like that 
```verilog
module timer_parameter_tb(

    );
    localparam FINAL_VALUE = 49_999;
    localparam BITS = $clog2(FINAL_VALUE);

    reg clk, reset_n, enable;
    wire done;
    
    // Instantiate module under test
    timer_parameter #(.FINAL_VALUE(FINAL_VALUE)) uut (
        .clk(clk),
        .reset_n(reset_n),
        .enable(enable),
        .done(done)
    );
    
    
    // Generate stimuli
    
    // Generating a clk signal
    localparam T = 10;
    always
    begin
        clk = 1'b0;
        #(T / 2);
        clk = 1'b1;
        #(T / 2);
    end
    
    initial #(FINAL_VALUE * T * 3) $stop;
    initial
    begin
        // issue a quick reset for 2 ns
        reset_n = 1'b0;
        #2
        reset_n = 1'b1;
        enable = 1'b1;
        
            
    end
    
endmodule
```