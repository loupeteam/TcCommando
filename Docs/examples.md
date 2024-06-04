## Examples:

Let's make an example program, to simulate an oven. We start with our `OvenBasic` function block:
```
FUNCTION_BLOCK OvenBasic
VAR_INPUT
END_VAR
VAR_OUTPUT
END_VAR
VAR
	_HeatCommand: PLCopenCommand;
	_Temp : REAL;
	_TempSetPoint : REAL;
END_VAR
```
Here, `_Temp` is representing the temperature of the oven, `_TempSetPoint` is the temperatuer which we want the oven to be at, and `_HeatCommand` is our `PLCopenCommand`, which will be used to trigger the oven to turn on. 

We have two properties, `Temp` and `TempSetPoint`, with trivial getters and setters for each variable.

We define the following methods for the function block:
```
METHOD RegulateTemp : BOOL
VAR_INPUT
END_VAR

---------------

//warm up/cool down over time
IF Temp < TempSetPoint THEN
	Temp := Temp + 1;
ELSIF Temp > TempSetPoint THEN
	Temp := Temp - 1;
END_IF

RegulateTemp := Temp = TempSetPoint;
```

The `RegulateTemp` function is a simple simulation of a real oven, for the purposes of this demo. It simulates an over changing its temperature by 1 unit towards its setpoint every cycle, until the setpoint is reached.

```
METHOD StartHeating : BOOL
VAR_INPUT
	status: IPLCopenCall;
END_VAR

---------------

IF status = 0 THEN
	_HeatCommand.Execute := 1;
ELSE
	status.Start(_HeatCommand);
END_IF
```
The `StartHeating` method is the API by which the heating is turned on. It takes an `IPLCopenCall` interface as an argument, which it then uses to start the heating command internal to the fub. Through this method, another function block or task can start a heat command and monitor the outcome. If no call object is passed (`status=0`), then we begin the heat command by setting its Execute bit high. If a call interface is passed, then we begin the command by calling the call's `Start` method, and giving it the oven's `_HeatCommand` as an argument.

```
METHOD Cyclic : BOOL
VAR_INPUT
END_VAR
VAR
END_VAR

---------------

IF _HeatCommand.Consume THEN
	TempSetPoint := 350; //set the oven temp set point
END_IF

RegulateTemp();

IF Temp = TempSetPoint THEN
	_HeatCommand.Finished();//report status when command is done
END_IF

_HeatCommand.Execute := 0;
```

This function is meant to be called cyclically by our program. The first `IF` statement is to monitor if the oven has been commanded to start. When a `PLCopenCommand` function block has been triggered, the `Consume` property gets set high until is read, to indicate that the new command has been started. 

Using this `OvenBasic` function block, we have program code as follows:

```
PROGRAM OvenExample
VAR
	oven : OvenBasic;
	heatCall : PLCopenCall;
	startHeating : BOOL;
END_VAR

---------------

oven.cyclic();

IF startHeating THEN
	oven.StartHeating(heatCall);
	startHeating := 0;
END_IF

heatCall();
heatCall.Execute := 0;

IF heatCall.Done THEN
	//do the next thing
END_IF
```
With this program code, we can set the bool `startHeating` to trigger the heating command. By passing an instance of a `PLCopenCall`, we can monitor its status separate from the status of the `OvenBasic` function block to see the progression of the command. When we set `startHeating := 1`, `heatCall.Busy` gets set high until the `OvenBasic` function block sets `_HeatCommand.Finished()`, at which point `heatCall.Done` gets set to 1. 
