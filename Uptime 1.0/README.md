# Sunrise server specification
***Version 1.0 (working draft)***

This document details a standard format for detailing server status and data, for applications/websites to fetch.

Note that this specification **does not** deal with its implementation, that and caching are left up to the server provider to grant greater flexibility in how its handled. What matters more is that everyone is on the same page, which this specification attempts to accomplish.

*Not to be confused with the `servers` section of the Manifest spec, although the two may merge/become interoperable in the distant future.*

## The XML file
The core of the `servers.xml` file looks like this:

```xml
<?xml version="1.0" encoding="UTF-8" ?>

<sunrise-status version="1.0" game="com.example.game1">
    <name>My Server</name>
    <updated>2019-05-01T13:00:00Z</updated>

    <servers>
        <!-- Add server blocks here. -->
    </servers>
</sunrise-status>
```

In the root tag, `version` should always equal the version of this specification, and the `game` should match the game ID used by both other servers of the same game, and optionally the Sunrise manifest if they use the same field there.

The root takes two fields, `name` and `updated` for the server group's name and when the status file was last updated.

## The server block
```xml
<servers>
    <server type="website">
        <name>Forums</name>
        <available value="true" last-available="2019-05-01T10:00:00Z" />
    </server>

    <server type="auth">
        <available value="true" last-available="2019-05-01T10:00:00Z" />
    </server>

    <server type="game">
        <name>Infinity</name>
        <available value="true" />
        <players current="500" max="1000" />
    </server>
</servers>
```

As few or as many `server` blocks can be added to the status manifest.

The server block accepts one attribute, **type**, which should equal either:
 - `website`: for websites, forums, wikis etc.
 - `auth`: for the game's authentication server.
 - `game`: for the shards that the game can connect to.

### Supported fields
 - `name`: *Recommended.* The user-facing name of the server. Will default to *Website*, *Auth server*, or *Game server* if unset.
 - `available`: *Required.* Takes two parameters.
   - `value`: *Required.* Whether the server is available & accessible.
   - `last-available`: *Optional.* ISO 8601 timestamp of when the server was last available.
 - `players`: *Optional.* For `type="auth|game"` servers only. Takes two attributes of `current` and `max` for current and maximum capacity.
