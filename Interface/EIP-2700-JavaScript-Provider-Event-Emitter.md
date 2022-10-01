# EIP-2700 JavaScript Provider Event Emitter
Provides the description of an object that is made available to JavaScript applications which they can use to receive notifications (Only describes the notification mechanism). 

## Interface
```
interface EthereumProvider {
	on(eventName: string, listener: (...params: unknown[]) => void): void
	removeListener(eventName: string, listener: (...params: unknown[]) => void): void
}
```
The specific events that can be listened to and the shape of their listener callback functions is left to be defined in separate standards.
* If `on` is called with an `eventName` that the provider is familiar with then the provider must call the provided `listener` when the named event occurs. 
* If the same `listener` is added multiple times to the same event via `on`, the provider may choose to either callback the listener one time or one time per call to `on`.
* If `removeListener` is called with an `eventName and listener` that was previously added via `on` then the provider must decrease the number of times it calls the `listener` per event by one.
