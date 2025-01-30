
In the 3rd phase of the implementation, you should implement the various events that occur in the traffic simulation. Do not proceed to the next phase until all tests for this phase pass.

**Event Class**

The `Event` class is an abstract class that represents an event in the simulation. It implements the `Comparable<Event>` interface, allowing events to be sorted based on their execution time.

```java
public abstract class Event implements Comparable<Event> {

    private static long _counter = 0;
    protected int _time;
    protected long _time_stamp;

    Event(int time) {
        if (time < 1)
            throw new IllegalArgumentException("Time must be positive (" + time + ")");
        else {
            _time = time;
            _time_stamp = _counter++;
        }
    }

    int getTime() {
        return _time;
    }

    @Override
    public int compareTo(Event o) {
        // implement
    }

    abstract void execute(RoadMap map);
}
```

An event is essentially an object with a specified execution time. When we implement the `TrafficSimulator` later, it will execute all events with time `t` at simulation time `t`. For now, the specific time is not important.

Executing an event `e` involves calling `e.execute(rm)`, where `rm` is the `RoadMap` object. The `RoadMap` will be provided by the simulator later. For our tests, we will use our own `RoadMap` instance.

**NewVehicleEvent**

This event represents the introduction of a new vehicle into the simulation.

* Constructor:

```java
public NewVehicleEvent(int time, String id, int maxSpeed, int contClass, List<String> itinerary) {
    super(time);
    // put the different values in fields
}
```

* `execute(map)` method:
    1. Create an itinerary as a `List<Junction>` from the itinerary provided to the constructor as a `List<String>`. Use the `map` to translate each junction ID to its corresponding `Junction` object.
    2. Create an instance `v` of `Vehicle` using the itinerary and other information passed to the constructor.
    3. Add `v` to the `map`.
    4. Ask `v` to move to the next road (it will then enter the first roal, and change its status from `PENDING` to `TRAVELLING`).

**Other New... Events**

The events `NewCityRoadEvent`, `NewInterCityRoadEvent`, and `NewJunctionEvent` are similar to `NewVehicleEvent`. They simply create simulated objects of different types. Note that `NewCityRoadEvent` and `NewInterCityRoadEvent` share common functionality, so you can create an abstract class `NewRoadEvent` to contain this common code.

**SetWeatherEvent**

This event sets the weather conditions on a road.

* Constructor:

```java
public SetWeatherEvent(int time, List<Pair<String, Weather>> ws) {
    super(time);
    //...
}
```

The list `ws` is a set of pairs `<id, w>`, where `id` is the identifier of a road and `w` is the weather condition to be set. Store the list `ws` in a field for use in the `execute` method.

* `execute(map)` method:
    * For each pair `<id, w>` in the `ws` list:
        * Find the road `r` corresponding to `id` using the `map`.
        * Call `r.setWeather(w)`.

**SetContClassEvent**

This event is similar to `SetWeatherEvent`, but it changes the contamination class of vehicles instead of the weather of roads.

**Testing**

Test your classes with the corresponding JUnit test classes.

