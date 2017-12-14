Take Home Assigment - TG17
====

# Preface
- I choose to implement the service using **Golang**, Go is a non-blocking IO language by nature and has a great conccureny model that fits well with the requirments of the system.
- I based the design on an internal [EventBus](https://github.com/asaskevich/EventBus), subscribing **Collectors** to the bus.
- Each **Collector** is responsible for fetching data from a remote API. _For instance:_ **DirectionCollector** fetch the directions data from google map api.
- The **Collector** is triggered asynchronisly (in a separate goroutine) only once, upon an **Event** (In the case of **DirectionCollector** upon **OrigDestEvent** that is being fired after processing the query params from the **/travel** API that the service exposes.
- **Collector** might also fire events, in the case of **DirectionCollector** it fires the **LatLongEvent** (after collecting this information from the json retrieved from google maps api) - this event triggers the **WeatherCollector** that retrieves the weather data from the weather remote api.
- Collectors passing the result on a dedicated Go channel called **Result Channel** (buffered channel), that channel is being observed by the http handler that gathers all the results together marshals them into a combined json and return the results to the client.

# Extensability
- The design supports extensability in a relaivly simple way.
- All of the collectors and events share the same interfaces.
- Each collector declares the event it listen to.
- The collectors are being registed in the **collectors_factory** and the subscription to the bus is done genericlly in the http handler for all the collectors in the factory.

## To Add a new Collector to the system one should
- Create the Collector and corresponding Event struct and implement the collector and event interfaces.
- Make sure to fire the event from the right place (where the event data is available for submission).
- Register the Collector in the **collectors_factory**.
- That's it.

# Code Quality and Testability
- The system supports timeout (http handler level) and graceful shutdown.
- Error handling is being managed where possible.
- [A Logger](https://github.com/sirupsen/logrus) was introduced to the system.
- The code is documented using internal comments.
- Runtime configuration is being supported in some areas (such as the remote API urls) using env variables.
- Tests were introduced for each Collector and for the http handler.
- There is a section in the tests that checks particular json attributes existence in the data being retrieved from the remote apis. The reason for that is to increase the probabilty of catching dangarous api changes and be able to response with hotfix that will keep backward compatability of the exposed API to the client.
- The service was monitored using [pprof](https://golang.org/pkg/net/http/pprof/) and there is an ability to turn it on in runtime using env variable.

# Misc
- For json manipulation [jsonq](https://github.com/jmoiron/jsonq) was being used.
- Transformation between Kelvin to Celsius was done using **unit query param** that is being supported by the weather remote api.
