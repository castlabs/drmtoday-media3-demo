# Android Media3 ExoPlayer DRMtoday integration

This repository hosts the DRMtoday Media3 ExoPlayer integration library. The library provides
an implementation of the `MediaDrmCallback` that can be used to send DRM license requests for
Widevine and PlayReady to DRMtoday.

You can create an instance of the callback as follows:

```
DrmtodayCallback drmtodayCallback = new DrmtodayCallback(
        httpDataSourceFactory, // The data source
        DrmtodayCallback.DRMTODAY_STAGING, // The Environment
        "client_dev", // The merchant
        "purchase", // The User ID
        "default", // The Session ID
        null, // The (optional) auth token
        null // The (optional) asset ID. For debugging purposes only. Overrides keyIds from DASH manifest
);
```

This will provide a statically configured instance that you can pass to your ExoPlayers
DrmSessionManager:

```
DrmSessionManager drmSessionManager =
        new DefaultDrmSessionManager.Builder()
                .setUuidAndExoMediaDrmProvider(drmSystemUuid, FrameworkMediaDrm.DEFAULT_PROVIDER)
                .build(drmtodayCallback);
```

Finally, you need to pass DrmSessionManager to the MediaSource that is used by the player:

```
player = new ExoPlayer.Builder(this)
        .setRenderersFactory(new DefaultRenderersFactory(this))
        .setTrackSelector(new DefaultTrackSelector(this))
        .build();

DataSource.Factory dataSourceFactory = new DefaultDataSource.Factory(this);
MediaSource videoSource = new DashMediaSource.Factory(dataSourceFactory)
        .setDrmSessionManagerProvider(unusedMediaItem -> drmSessionManager)
        .createMediaSource(MediaItem.fromUri(Uri.parse("https://content.players.castlabs.com/demos/drm-agent/manifest.mpd")));

player.prepare(videoSource);
```

If you need to re-use the player instance for different content, you can also dynamically
configure the callback:

```
// Create a callback instance
DrmtodayCallback drmtodayCallback = new DrmtodayCallback(
        httpDataSourceFactory // The data source
);

// Since the DRM configuration is content specific, we need to
// configure the DRMtoday callback accordingly before we load content into the player
drmtodayCallback.configure(
        DrmtodayCallback.DRMTODAY_STAGING, // The Environment
        "client_dev",       // The merchant
        "purchase",    // The User ID
        "default",   // The Session ID
        null, // The (optional) auth token
        null // The (optional) asset ID. For debugging purposes only. Overrides keyIds from DASH manifest
);
```

## ExoPlayer versions

The demo application in this repository is using ExoPlayer version 3.1.1.


## License

The library is released under the [Apache 2.0] (http://www.apache.org/licenses/LICENSE-2.0) license.