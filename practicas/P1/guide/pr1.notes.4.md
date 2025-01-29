In the 4th phase of the implementation, you should implement the `TrafficSimulator` class. This class allows you to add and execute events, as well as advance the state of the simulated objects that were added by the events. Do not move on to the next phase until all tests for this phase are passing.

**Traffic Simulator**

```java
public class TrafficSimulator {

    RoadMap _roadMap;
    Queue<Event> _events;
    int _time;

    public TrafficSimulator() {
        _roadMap = new RoadMap();
        _events = new PriorityQueue<>();
        _time = 0;
    }

    //...
}
```

For the `_events` queue, use a `PriorityQueue` so that you can extract events in the order of their time (and timestamp).

* `reset()`:
    * Clears the `_roadMap` and `_events`, and sets the `_time` to 0.

* `addEvent(e)`:
    * Throws an exception if the time of event `e` is less than or equal to the current time (`_time`), as we cannot add events to the past.
    * Adds event `e` to the `_events` queue.

* `advance()`:
    * Increments the `_time` by 1.
    * Executes all events in `_events` that have a time equal to `_time`, and removes them from `_events`. This should be done efficiently, without traversing the entire queue. You should only process the events at the beginning of the queue that have the correct time. Remember that events are dequeued in the desired order because of the priority queue.
    * Calls the `advance()` method of all junctions in the `_roadMap`, and then calls the `advance()` method of all roads in the `_roadMap`. The order is important: junctions must be advanced before roads. The `advance()` method of vehicles is not called directly here; it will be called by the corresponding roads.

* `report()`:
    * Creates a JSON object as explained in the PDF. You should use the `_roadMap.report()` method to generate the JSON for the roadmap.

**Testing**

Test your implementation with the `TrafficSimulatorTest` JUnit test class.

