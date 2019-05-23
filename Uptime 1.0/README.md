# Sunrise server specification
***Version 1.0 (working draft)***

This document details a standard format for detailing server status and data, for applications/websites to fetch.

Note that this specification **does not** deal with its implementation, that and caching are left up to the server provider to grant greater flexibility in how its handled. What matters more is that everyone is on the same page, which this specification attempts to accomplish.

*Not to be confused with the `servers` section of the Manifest spec, although the two may merge/become interoperable in the distant future.*

## The XML file
To avoid confusion, it is recommended (but not required) that the XML file hosted by your server uses the `.status.xml` extension. For example, information from "OpenShips" servers may be accessible from https://openships.net/servers.status.xml.

The core of the XML file looks like this:

```xml
<?xml version="1.0" encoding="UTF-8" ?>

<sunrise-status version="1.0" game="com.example.starships">
    <name>OpenShips Servers</name>
    <updated>2019-05-01T13:00:00Z</updated>

    <servers>
        <!-- Add server tags here. -->
    </servers>
</sunrise-status>
```

On the root **sunrise-status** block, the version should always equal this specification's version number. The **game** attribute should be the same across all projects that target the same game - this way anything that fetches this data knows to only accept manifests for "game 1" and not "game 2". It is *not* for your server/project name, just the game it targets.

The root takes two tags, `name` and `updated` for the server group's name and when the status file was last updated (in ISO 8601 time).

## The server tag
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
        <name>Liberation</name>
        <available value="true" />
        <players current="500" max="1000" />
    </server>
</servers>
```

As few or as many `server` tags can be added to the status manifest.

The server tag accepts one attribute, **type**, which should equal either:
 - `website`: for websites, forums, wikis etc.
 - `auth`: for the game's authentication server.
 - `game`: for the shards that the game can connect to.

### Supported fields
 - `name`: *Recommended.* The user-facing name of the server. Will default to *Website*, *Auth server*, or *Game server* if unset.
 - `available`: *Required.* Takes two attributes:
   - `value`: *Required.* Whether the server is available & accessible.
   - `last-available`: *Optional.* ISO 8601 timestamp of when the server was last available.
 - `players`: *Optional.* For `type="auth|game"` servers only. Takes two attributes of `current` and `max` for current and maximum capacity.
