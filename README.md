
===============================================================================

    This is a simple 8 input interrupt controller.
    Written in verilog.

    Currently it supports two modes, polling and custom priority.

    In polling, there are no priorities. All the interrupts are
    checked periodically and if any one of them is active then it
    is serviced.
 
    In the custom priority mode, the priorities are set during the
    initialization phase. The controller then polls the sources in
    this custom order.

Operation
===============================================================================
    After Reset, the controller waits for commands from the master
    (processor) on the bus. The lowest 2 bits determine the mode of
    operation.

    01 - Polling mode
    10 - Priority mode.

    It keeps waiting for valid input mode.

Polling
===============================================================================
    The bus is driven as xxxx_xx01 for exactly 1 clock by processor.
    Controller knows this is the polling mode. It then enters the
    polling state where it keeps checkig for all interrupt sources
    in cycle.

    If any interrupt is found active then intr_out is set to 1.
    After that, the controller waits for an acknowledgement from
    the processor.

    Once the processor gives acknowledgement, exactly for 1 clock
    cycle on intr_in by a high -> low -> high transition, the
    controller starts driving the bus with the condition code of
    01011_intrID. The intrID part is the ID of the source currently
    being serviced.

    Controller keeps this data on the bus it it receives another
    acknowledgement from the processor on the intr_in pin in the
    same mannter (High -> Low -> High).

    After that, the controller waits for the confirmation from the
    processor that the interrupt has been serviced. Processor, once
    done, sends another acknowledgement on the intr_in pin in the
    same mannter (High -> Low -> High) along with the condition
    code of 10100_intrID on the bus. This interrupt ID must match
    the one sent last by the controller. 

    If either the condition code or the intrID does not match then
    the controller goes to reset state. Else it just checks (polls)
    the next source and continues in this cycle unless reset.


Custom Priority
===============================================================================
    Working of this mode is exactly similar to the polling exept
    that it does not poll the sources in order.

    After reset, if the input is xxxyyy10 then the controller is
    in custom priority mode. xxx is the source ID of the highest
    interrupt and yyy is the ID of the second to highest. The
    processor needs to give 4 such cycles to and give the prio-
    -rities for all 8 sources.

    After that the controller goes on checking from the highest
    to lowest source to check if any of them is active. If it is
    then the operation continues similar to the polling mode.

    The only things that change are the condition codes that are
    sent to and from the processor. The controller sends 10011
    and the processor acknowledges with 01100.

-------------------------------------------------------------------------------
TODO:   Currently there is no way of checking if two (or more) of
        the sources are same. This can be good if you want to disable
        some interrupts. You can repeat the last priority till you
        have reached the total count of 8.
-------------------------------------------------------------------------------

Timing Diagram
===============================================================================

intr_out         __________________
            ____|                  |___________________________________________

intr_in     __________________      ______________      ___________      _____
                              |____|              |____|           |____|

data_bus    _______________________|||||||||||||||||||||___________||||||_____
           

-------------------------------------------------------------------------------
    Note - The first time the data_bus is active is when the controller
    drives the bus. Next time when it's active, the processor drives it.
-------------------------------------------------------------------------------
    Note - The timing diagram remains the same on both polling and custom
    priority modes. Only thing that changes is the ack data on the bus.
-------------------------------------------------------------------------------

Condition Codes
===============================================================================

    Polling:
        From Controller     -   01011
        From Processor      -   10100

    Custom Priority
        From Controller     -   10011
        From Processor      -   01100

===============================================================================

