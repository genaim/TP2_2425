In the 5th phase of the implementation, you should implement the `Controller` and `Main` classes. The `Controller` class manages the interaction between the user and the simulator, loading events from a JSON file and passing them to the simulator. The `Main` class is the entry point of the application. Do not proceed to the next phase until all tests for this phase pass.

**Controller Class**

The `Controller` class has the following constructor:

```java
public class Controller {
    private TrafficSimulator _sim;
    private Factory<Event> _eventsFactory;

    public Controller(TrafficSimulator sim, Factory<Event> eventsFactory) {
        _sim = sim;
        _eventsFactory = eventsFactory;
    }
    //...
}
```

As you can see, the constructor takes an instance of `TrafficSimulator` and a factory to create events, and stores them in the corresponding fields.

The `Controller` class has the following methods:

* `loadEvents(InputStream in)`:
    * This method loads events from an input stream (e.g., a file) into the simulator. The input stream is assumed to contain a JSON object in the following format:

    ```json
    { "events": [e1,..., en] }
    ```

    where each `ei` is a JSON object representing an event.

    * The method first builds a `JSONObject` from the input stream:

    ```java
    JSONObject jo = new JSONObject(new JSONTokener(in));
    ```

    * Then, it iterates over the list of events in `jo.getJSONArray("events")`, uses the `_eventsFactory` to create the corresponding event objects, and adds them to the `_sim` traffic simulator.

* `run(int n, OutputStream out)`:
    * Executes the simulator for `n` steps and writes the output to the given output stream.
    * To facilitate writing to the output stream, you can create a `PrintStream`:

    ```java
    PrintStream p = new PrintStream(out);
    ```

    and then use `p.println(...)` to write to the output stream.

    * The output should be a JSON object in the following format:

    ```json
    { "states": [s1,..., sn] }
    ```

    where each `si` is a JSON object representing the state of the simulation at step `i`.

    * You can generate the output as follows:

    ```java
    // print "{" to 'p'
    // print "  \"states\": [" to 'p'

    // loop for the first n-1 states (to print comma after each state)
    for (int i = 0; i < n - 1; i++) {
        _sim.advance();
        p.print(_sim.report());
        p.println(",");
    }

    // last step, only if 'n > 0'
    if (n > 0) {
        _sim.advance();
        p.print(_sim.report());
    }

    // print "]" to 'p'
    // print "}" to 'p'
    ```

* `reset()`:
    * Simply calls the `reset()` method of the `_sim` traffic simulator.

**Main Class**

The `Main` class is the entry point of the application. It should do the following:

1. Add a command-line option `-t n` to specify the number of steps to simulate. Store the value in a static field, e.g., `_timeLimit`. If the `-t` option is not provided, use a default value for `_timeLimit`.

2. Fill in the code of the `initFactories` method to initialize the factories for events, light switching strategies, and queuing strategies. Assign the events factory to the `_eventsFactory` field.

3. Fill in the code of the `startBatchMode` method as follows:

    * Create an `InputStream` from the input file specified by the `_inFile` field.
    * Create an `OutputStream` from the output file specified by the `_outFile` field. If `_outFile` is null, use `System.out` as the output stream.
    * Create a `TrafficSimulator` object.
    * Create a `Controller` object, passing the `TrafficSimulator` and the events factory to the constructor.
    * Load the events from the input stream into the controller using the `loadEvents` method.
    * Close the input stream.
    * Call the `run` method of the controller with `_timeLimit` as the number of steps and the output stream as the output destination.

**Testing**

Test your implementation with the `MainTest` JUnit test class.
