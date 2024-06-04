## `PLCopenCommand`
`Command` FUB, used to trigger an action and consume the status response. Intended use is that a component will have an instance of a `PLCopenCommand`. For the component implementation with a `PLCopenCommand` named `myCommand`, the pattern is as follows:
- Cyclically monitor `myCommand.Consume`. If `True`, then begin command implementation.
- When implementation is finished/errored/aborted call `myCommand.Finished`, `myCommand.Errored` or `myCommand.Aborted` respectively.

To trigger the command, call the `Execute` property or the `ExecuteWithStatus` method. The `Execute` property can be called from a watch window and will start the command without reporting its status to any `remote`. In contrast, the `ExecuteWithStatus` method takes an instance of a `remote`, which will receive updates to its status as the command progresses. This is the method called internally by a `PLCopenCall` to begin a command and monitor its status.


## `PLCopenCall`
`Call` FUB, used to begin a `command` receive status updates. To use the `PLCopenCall` FUB, call the `Start` method and pass a `PLCopenCommand` FUB. This will begin the command by calling its `ExecuteWithStatus` method. The command status can be monitored by reading the `Busy`, `Error`, and `Done` properties of the `PLCopenCall` FUB. 

## `PLCopenCommandGroup`
`Command` FUB, used to bind multiple actions together into one call. It functions like a `PLCopenCall` with the same API, with the exception of the `Subscribe` and `Unsubscribe` methods, which take a `PLCopenCommand` FUB as an argument. When the command group is executed, all commands which are subscribed to it are executed. The status of the command group is determined by aggregating the statuses of all subscribed commands: If any have an error status, then the group is errored. If some are busy and none are errored then the group is busy, and if none are busy or errored then the group is done. If this command group is executed again, all subscribers will execute again.


## `PLCopenCallGroup`
FUB used to call multiple `PLCopenCommand` FUBs and monitor their status. This FUB is used by calling the `Start` method and passing a `PLCopenCommand` FUB. This will then call the `Start` method of the `PLCopenCall` FUB in the call group's internal array of `PLCopenCall` FUBs. The group completion status is determined similarly to the `PLCopenCommandGroup`. This structure is intended to start multiple calls and monitor them together as the progress, in contrast to the `PLCopenCommandGroup` which is used to group commands for calling multiple times. Setting the `Execute` property high does _not_ call all of the existing calls again, rather it starts a new instance of the Command in the `Command` property.  


## `PLCopenExclusiveGroup`
Used to control commands that are mutually exclusive. QUESTION: Is this implemented? Why does `RegisterCommands` take a bool?

## `PLCopenRemote`
`Remote` FUB, represents an object interested in the status of a `command`. The `command` uses the `remote` as an interface to report its progress to. This FUB is considered internal and likely will not need to be used in application logic.

## `PLCopenNullRemote`
FUB used to specify that there is remote. For example, if an command is triggered internally it should have a null remote, since there was no remote trigger that needs to monitor its status. This FUB is considered internal and likely will not need to be used in application logic.

