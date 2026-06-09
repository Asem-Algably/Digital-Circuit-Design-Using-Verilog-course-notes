In this module we discuss some guidelines for writing behavioral modeling, some of these rules are stylistic and others aren’t

- Use blocking (immediate) assignments for combinational circuits
- Use nonblocking (deferred) assignments for registers
- Assign a variable only in a single always block
- Separate memory components (registers) into individual code segments (normally next state logic, registers, and output logic), following this rule might generate verbose code but it would be very robust and easy to debug

>[!TIP]
> a pattern that is really useful and you might make use of is to define two registers inside of the module implementation, let’s assume `Q_reg` and `Q_next`, and use them in your code to represent the next value of the flip flop and the current value, this is suppose to make your code easier to work with, but if you don’t understand what I mean skip to the next module and notice their usage pattern in the code