# Local Dictation

Local Dictation is a private, fully local replacement for macOS Dictation. It turns a user’s speech into text in the currently focused application without sending audio or transcripts to a remote service.

## Language

**Dictation request**:
An accepted user instruction to begin dictating. A request may wait for the speech runtime to become ready before a session starts.
_Avoid_: Wants listening, pending start

**Dictation session**:
The period from microphone capture beginning until the transcript is finalized or the user cancels. A request can exist without a session, but only one session may exist at a time.
_Avoid_: Recording, connection

**Speech runtime**:
The local speech-recognition capability that holds the model in memory and produces transcript updates. It may be ready or dormant independently of whether a dictation session exists.
_Avoid_: Backend, engine, Python server

**Speech provider**:
The chosen speech-recognition model behind the speech runtime. Selecting a different provider replaces the running speech runtime; it does not affect an accepted dictation request's meaning.
_Avoid_: Backend, mode

**Draft transcript**:
A transcript update that supersedes everything produced so far in the session and may revise earlier words. Distinct from an appended update, which only adds to what was already inserted.
_Avoid_: Partial, interim, delta

**Ready**:
The speech runtime is available to begin a dictation session without loading the model again.
_Avoid_: Idle, connected

**Dormant**:
The speech runtime has been intentionally unloaded to release memory. A dictation request moves it through warm-up before a session can begin.
_Avoid_: Idle, dead, failed

**Warm-up**:
The period in which a dormant or newly launched speech runtime is becoming ready. An accepted dictation request remains pending throughout warm-up unless the user withdraws it.
_Avoid_: Reconnect, generic starting

**Finalization**:
The period after microphone capture stops while trailing transcript text is produced and delivered. Finalization is part of the dictation session.
_Avoid_: Processing, stop

**Cancellation**:
User-requested immediate termination of a dictation session without waiting for trailing transcript output. Text already inserted remains as-is.
_Avoid_: Stop, finalization

**Interruption**:
Unexpected termination of a dictation session because the speech runtime became unavailable. Inserted text remains and a new user request is required after recovery.
_Avoid_: Cancellation, automatic resume

**Terminal target**:
A focused application where an inserted line break could execute a command. Transcript text is still inserted live, with line breaks and tabs collapsed to spaces so no command is submitted mid-dictation.
_Avoid_: Shell mode, console, buffered insertion

## Example dialogue

> **Developer:** A dictation request arrived while the speech runtime was dormant.
>
> **Domain expert:** Keep the request pending during warm-up. Start the dictation session only when the runtime is ready.
>
> **Developer:** The user released the hold key before warm-up completed.
>
> **Domain expert:** Withdraw the request; no session began, so there is nothing to finalize.
>
> **Developer:** The user pressed Escape while finalization was in progress for a terminal target.
>
> **Domain expert:** Cancel the session. Text already inserted stays; no line break is ever inserted into a terminal target.
>
> **Developer:** The speech runtime failed during a different session.
>
> **Domain expert:** That session was interrupted. Preserve inserted text and wait for a new dictation request after recovery.
