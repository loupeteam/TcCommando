# TcCommando
A wrapper for using a PLCopen-like interface with sequencers, functions, and more.

## Version Information
This documentation is for version `0.01.0` of the TcCommando library.

## Overview
TcCommando provides a way to abstract and monitor an object's commands in your TwinCAT solution. TcCommando decouples function calls and response monitoring from implementation. This enables remote function calls, multiple instances of function calls with separate status monitoring, and ways to group calls together and manage their status as a group. 
### Why Do Anything?
Decoupling calls, commands and implementation code becomes more useful as an application grows in size and complexity. 

Imagine an axis with input commands `Enable`, `Power`, `MoveAbsolute`, `MoveAdditive`, `MoveContinuous` and output statuses of `Busy`, `Done` `Error`. In a simple case, one could keep track of which command the output status bits are in reference to. What happens however when the `MoveAbsolute` input gets set high when the axis is already in the middle of a `MoveAdditive` command? If multiple commands are triggered it is suddenly no longer clear what the output bits of the function block mean. Even if a convention is established e.g. the first command is the one the function block responds to, all subsequent commands have no status information. While the likely case is for a conflicting command to immediately resolve to error or done, having that information separate and accessible to where it was called helps maintain clarity on the machine state. Even if the FUB has status outputs specifically for each command, the same problems arise when the same command is triggered more than once. 

For a higher level example, imagine a pick-and-place robot with process code and an HMI, both of which can trigger a move. In this case, both systems which can trigger a move have an interest in monitoring the progress of their individual command. If the API has called a move and an operator attempts another move from the HMI, each call must be monitored separately to keep each subsystem synchronized and aware of how their command is progressing. If the robot simply had a `Busy`, `Done` and `Error` status it wouldn't possible to differentiate which command the status was in reference to. Another benefit to this architecture is that adding multiple API endpoints, HMI connections or new objects to command the robot doesn't require changing any of the status information of the robot interface. This modularity allows maximum flexibility in adjusting the application logic and cross-process communication.

## Terminology

This section defines some of the common terms that are used throughout the TcCommando documentation.

- An implementation is object code which performs a function internally to a function block. It is intended to be invoked externally, but controls a process internal to a FUB.
- A `command` is the API by which the object implementation is invoked. The object monitors the `command` to know when to begin, and it updates the `command` with its status as it progresses. A `command` can be started internally, or invoked remotely using a `call`. 
- A `call` is the API for a `command`. It handles starting a `command`, and the `command` reports its status back to the `call` so that the caller can monitor the status of the command. A call need not be invoked from the FUB which does the implementation, and there can be more than one instance of a `call` for a given command. 

For details and specific uses, see the [Function block explanation page](FUBs.md).

## Use Cases

- Machines with process logic that is distributed across several tasks or modules
- Machines that can benefit from "loose coupling" of subsystems
- Machines that require "1 to many" or "many to 1" connections between subsystems
- Machines that have a process that requires steps to be performed in order
- Machines that have optional steps in a process
- Machines that need a configurable process

## Overview

[Function block descriptions](Docs/FUBs.md)

[Examples](Docs/examples.md)