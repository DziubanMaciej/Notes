parentTopic:: [[Computer]]

## Proceeding the simulation
Processes do not run until we proceed the simulation with `sc_start()`. We can pass a certain time value, e.g. `sc_start({30, SC_NS})` to run the simulation for the specified amount of time. Calling `sc_start()` without parameters will run it until stopped. Note: exiting from a process doesn't mean simulation is stopped (it could be started). You have to call `sc_stop()`. 

The simulation cannot be restarted then - `sc_start()` cannot be called after `sc_stop()`. This makes it hard to test SystemC with frameworks like gtest. Every testbench should probably be a separate binary (or just run in a separate process).



## Processes
Processes are not hierarchical - they cannot call each other. But they can cause other processes to execute by assigning new values to signals in the sensitivity list of the other process. There are three main types of processes.

| Methods                                                                                                 | Threads                                                                                              | Clocked threads                                                           |
|---------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------|
| Can handle simple logic, e.g. combinatorial operations. Lasts only one cycle and cannot be interrupted. | Can handle more complex logic over multiple clock cycles.                                            | Can handle more complex logic over multiple clock cycles.                 |
| Synthesizable                                                                                           | Not synthesizable                                                                                    | Synthesizable                                                             |
| Defined with `SC_METHOD(methodName)` in constructor                                                      | Defined with `SC_THREAD(threadName)` in constructor                                                    | Defined with `SC_THREAD(cthreadName, anEvent)` in constructor                |
| Sensitivity list defined with `sensitive` field                                                         | Sensitivity list defined with `sensitive` field                                                       | Can be sensitive to only one single-bit event specified during definition |
| Dynamic sensitivity with `next_trigger()`                                                                | Dynamic sensitivity with `wait(int, sc_time)` or `wait(sc_event)`                                      | No dynamic sensitivity                                                    |
| `wait()` is illegal - method cannot be interrupted                                                      | `wait()` can suspend the thread until an event from sensitivity list is triggered                    | `wait()` is allowed, but no dynamic sensitivity                           |
| Reset unsupported                                                                                       | Can be reset with `reset_signal_is()`                                                                 | Can be reset with `reset_signal_is()`                                      |
| Called once during initialization and then according to sensitivity                                     | Start immediately on `sc_start()` in an undefined order. It can be disabled with `dont_initialize()`   | Start only after first event trigger                                     |

### Static sensitivity
Processes can be sensitive to events. Every time an event from the sensitivity list is notified, the process will run.
- methods and threads can have any number of events they are sensitive to. The list is defined in constructor after defining the process
  ```c++
  SC_METHOD(myMethod)
  sensitive << clock.pos() << inputA << inputB;
  ```
- clocked threads are sensitive to only one event specified during definition

When using clock signal for sensitivity list, there are 3 options

- Can be level-sensitive - triggered at both edges: `sensitive << clock`
- Can be posedge-sensitive: `sensitive << clock.pos()`
- Can be negedge-sensitive: `sensitive << clock.neg()`

### Dynamic sensitivity
Threads (not clocked threads) can have dynamic sensitivity - specified during `wait()`. They can either wait on any arbitrary event not specified on the sensitivity list or wait for some amount of time, e.g. `wait(10, SC_NS)`.

Methods can change their sensitivity for one invocation using `next_trigger()` method. It has a couple of overloads. If it's called multiple times, the last one counts. When dynamic sensitivty is used, static sensitivty is ignored. It is restored upon next method invocation.
- `next_trigger()` - reset to static sensitivity
- `next_trigger(someEvent)` - be sensitive to an event.
- `next_trigger(10, SC_NS)` - call this method after a specified amount of simulation time
- `next_trigger(10, SC_NS, someEvent)` - call either after a specified time or after an event notification

### Resetting threads
Threads and clocked threads can be reset by defining a reset signal with `reset_signal_is(signal, true)` (in constructor, after thread definition). When the `signal` becomes `true` the thread will be stopped and started from the beginning. Programmer should take care to include some initialization code at the beginning of the thread to ensure it works in the same way after resetting. Also, remember to set the reset signal for only one clock cycle. Otherwise, the process will keep resetting each cycle.

```c++
void threadName() {
    // Reset all the state
    wait();

    while(true) {
        wait();
        
        // Do stuff
    }
}
```

## Tristate

## Waveform tracing
For some reason calling `sc_start()` without parameters causes the trace to be empty. Always call it with some concrete timeout.
```
sc_trace_file* trace = sc_create_vcd_trace_file("myTrace");
sc_trace(trace, clock, "Clock");
sc_start({30, SC_NS});
sc_close_vcd_trace_file(trace);
```
Open trace with `gtkwave myTrace.trace`.


## Implementing testbenches
Testbench typically define some test module to provide stimulus and/or validate DUT (device under test) outputs. For all ports we have to define an `sc_signal` and bind the signals to ports. SystemC will verify if all ports were bound.

Do not try to read and write the signals directly from `sc_main`. It is technically possible, but has different characteristics than interacting with DUT through an actual `SC_MODULE`. Always define a test module and use it. The `sc_main` function should only create signals, bind them, optionally create a [[SystemC#Waveform tracing|vcd trace]] and call `sc_start`.

Order of process execution is unspecified, so sometimes our test module can run before or after the DUT in a given cycle. In order to be sure we're reading values for verification from a correct cycle, we can call `wait(SC_ZERO_TIME)` before the verification. This will schedule a delta cycle wait and ensure verification happens **after** all other process run (unless they called this wait too, but DUTs generally shouldn't do it).

Example of a SystemC test:
### DUT
Device under test will be a simple 4-bit binary counter
```c++
SC_MODULE(Dut) {
    sc_in_clk clock;            // clock input
    sc_out<sc_uint<4>> counter; // output port
    SC_CTOR(Dut) {
        SC_CTHREAD(main, clock.pos());
    }
    void main() {
        while (1) {
            wait();
            counter = (counter.read() + 1) % 4;
        }
    }
};
```

### Tester
The tester module will wait for the same clock and check the output. It could also be writing to DUT's output, if it had any more than the clock.
```c++
SC_MODULE(Tester) {
    sc_in_clk clock;           // clock input
    sc_in<sc_uint<4>> counter; // this will be bound to Dut's output. Direction to be reversed: sc_out becomes sc_in and vice versa
    SC_CTOR(Tester) {
        SC_THREAD(main); // better to use thread instead of cthread for testers. They allow more functionality, e.g. delta cycle waits
        sensitive << clock.pos();
    }
    void main() {
        wait(1);
        wait(SC_ZERO_TIME);
        assert(counter.read() == 0);

        wait(1);
        wait(SC_ZERO_TIME);
        assert(counter.read() == 1);

        wait(1);
        wait(SC_ZERO_TIME);
        assert(counter.read() == 2);

        wait(1);
        wait(SC_ZERO_TIME);
        assert(counter.read() == 3);

        wait(1);
        wait(SC_ZERO_TIME);
        assert(counter.read() == 0);

        wait(1);
        wait(SC_ZERO_TIME);
        assert(counter.read() == 1);
    }
};
```

### Main function
```c++
int sc_main(int argc, char *argv[]) {
    Dut dut{"dut"};
    Tester tester{"tester"};

    // Create signals and bind to both DUT and Tester
    sc_signal<sc_uint<4>> value;
    dut.counter(value);
    tester.counter(value);

    // Create clock and bind to all components
    sc_clock clock("clk", 2, SC_NS, 1, 0, SC_NS, false);
    dut.clock(clock);
    tester.clock(clock);

    // Run simulation for enough time, so the tester can test everything
    sc_start({20, SC_NS});
    return 0;
}
```



## Deprecated stuff
SystemC is quite old and a lot of tutorials suggest old-style syntax. This chapter gathers these techniques and propose up-to-date solutions
- `sc_start(10)` - here the time unit is implicit (set with `sc_set_time_resolution`). Currently this overload is gone and we have to use `sc_start(10, SC_NS)`.
- Waiting on event in clocked threads. Only allowed waits in `SC_CTHREAD` are `wait()` and `wait(numberOfCycles)`. Only `SC_THREAD` can to `wait(anEvent)`.
- The `sensitive_pos` and `sensitive_neg` fields in `SC_CTOR`. Use `sensitive << clock.pos()` instead.
- `myModule.somePort(clock.signal())` during binding clock to a port. Simply pass the clock instead: `myModule.somePort(clock)`.
- `wait_until` and `delayed`. Not sure what the alternative is.
