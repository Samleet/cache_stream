
# Cache Stream
This plugin helps cache data on your apps and stream them when requested, even when offline thus increasing your app speed 10x faster and improving load time.

You can use plugin to cache any kind of data types supported by the dart framework.


## Features
* Cache data (any type) for offline or faster rendering.
* Apply mounting rules to validate each stream being mounted.
* Unmount / flush streams if no longer needed from cache.
* (Optional) Set refresh rate so stream is updated at interval.
* Force refresh stream to override from cache.


## Usage
To use this plugin, add `cache_stream` as a [dependency in your pubspec.yaml file](https://plus.fluttercommunity.dev/docs/overview).

### Examples
Import the library.

```dart
import 'package:cache_stream/cache_stream.dart';
```

Mounting the stream.

```dart
CacheStream cacheManager = CacheStream();

//Supply a Map<S,M<S,D>> of mounted entry to the caching stream
cacheManager.mount({
    'flights': {
        'data': () async {
            return await FlightController.index();
        },
        //optional
        'rules': (data){
            return data['flight'].isNotEmpty;
        },
    },
    'tickets': {
        'data': () async {
            return await TicketController.index();
        },
        'rules': (data) => data.isNotEmpty,
    }

    //refresh every 60 seconds after the initial mount | null 
    //you can omit Duration or set to: null for just onetime

}, Duration(seconds: 60));
```

Fetching the stream.

```dart
final flights = await cacheManager.stream(
    'flights', 
    fallback: () async {
        //incase stream isn't found, network error or has error
        return FlightController.index();
    },
    //optional
    callback: (data) {
        //We have the data - so let's transform the final result
        return flightToModel(data);
    }
);
```

If you ever need to force override the cache to stream fresh data then supply a `refresh` flag to the method. This will attempt to remount the stream from the source earlier specified during mount.

```dart
final flights = await cacheManager.stream(
    'flights', 
    refresh: true,
    //...
);
```

Unmounting streams.

There are basically 3 ways to unmount or flush the streams from memory.

```dart
//unmount only - stop refresh ticker if set on the mounted call 
cacheManager.unmount();

//unmount and flush all the data streams that have been mounted
cacheManager.unmount({});

//unmount and flush specific stream that was previously mounted 
cacheManager.unmount({
    'flights',
    'tickets'
});
```

## Heads up!

CacheStream will throw a fatal exception when attempting to cache dart objects. This is an expected behaviour from Dart. To override this issue - You need to define a `toJson` method on any mount call that fetches or has an object model among it returned resource.

```dart
//flight.dart

//...
Map<String, dynamic> toJson() => {
    'id': id,
    'name': name,
    'departure': departure,
    'planeId': planeId,
    'organization': organization
};
```

## Limitations

The plugin cannot mount a single stream with N(+1) depth. This means that if 2 identical stream is mounted with a "timer", only the last mounted stream is watched (i.e This is to avoid threading the app with multiple time tickers). 

So let's say we have stream: "flights" mounted on "flightScreen.dart" like this:

```dart
cacheManager.mount({
    'flights': {
        //...
    },
}, Duration(seconds: 60));
```

and we also have the same stream mounted on "ticketScreen.dart" as well but with a different refresh timer specified.

```dart
cacheManager.mount({
    'flights': {
        //...
    },
}, Duration(seconds: 30));
```
Each time you swap from either of the screens - The Timer will remount for the current screen by issueing 2(+) total refresh timer for one stream (flights) which might become an overhead or cause lags for your app depending on the stream context (e.g API call or heavy Future's)

To avoid this issue - it's advisable to use unique names for a stream. But if you really need to mount the stream again (probably to get new data before later fetching it) - then DON'T SET a refresh timer on the mount or use the `refresh` flag when calling a stream as stated above.

```dart
cacheManager.mount({
    'flights': {
        //...
    },
} /* Duration(seconds: 30) */); //<= remove the Duration
```

## Thank You! 😉

This is my first plugin on pub.dev - hopefully you find it useful for your project. if you find any issues, please don't hesitate to file a new issue via the issue tracker.

I'm also open for collaboration on exciting projects. please leave a like to support this project as i am developing this on my spare time.