verilog has different types of assignments like
- **continuous assignment**: this is used to set the output in the dataflow modeling, it basically says that the left hand side is continuous updated as the right hand side is changing, we use it with the `assign` keyword
- **procedural assignment**: this is the assignment method used in an always statement, the left hand side doesn’t change when the right hand side gets changed but when a variable in the sensitivity list gets triggered

### blocking and non blocking procedural assignment
there are two types of procedural assignment
#### blocking assignment
```verilog
module some_circuit (
		// inputs and outputs
	);
	always @( * )
	begin
		y = (a & b) | c;
	end
endmodule
```
blocking assignment code blocks the code after it and executes what is after it only after the current had been completed
the order of the assignment statements here is very important because it blocks the code after it which might generate different circuits depending on which statement got evaluated first

#### non blocking assignment
```verilog
module some_circuit (
		// inputs and outputs
	);
	always @( * )
	begin
		y <= (a & b) | c;
	end
endmodule
```
non blocking assignments shouldn’t be used with combinational circuits because it requires a feedback signal
non blocking assignments schedules the code assignment at the end which doesn’t block the code after it so all of the code runs simultaneously 
once we define the same output multiple times in non blocking assignment, we lose the first assignments since the last one overrides them

### blocking and non blocking in sequential circuits 
actually most of the time in sequential circuits you shouldn’t use blocking assignment, if you were designing only one storage element this might be fine but once you have more you should only use non blocking

>[!NOTE]
> the key takeaway here is use blocking assignment while working on combinational circuits, and non blocking assignments while working on sequential circuits

