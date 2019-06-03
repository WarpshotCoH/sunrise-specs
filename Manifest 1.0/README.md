# Sunrise manifest specification
***Version 1.0 (working draft)***

The Sunrise manifest format provides a clean separation of different client projects from both each other and the original base game. This is done by splitting projects into "applications" and "runtimes", partially inspired by the Flatpak and Snap packaging systems on Linux (though much less complex). Servers are also split into a separate area, to allow easier distribution and access to custom servers that can share application code.

> ***Example game/servers***
>
> To illustrate how all the pieces of Sunrise work together, through the example XML snippets in this file we are going to assume that:
> - The game Sunrise is targeting is called "Starships", published by "ExampleCorp". Its original website before shutdown was http://starships.example.com, so for the root game ID we would use `com.example.starships`.
> - Starships has one major expansion, "Lightspeed". This expansion was universally beloved *(ha)*. Its runtime ID would therefore be `com.example.starships.lightspeed`.
> - A group of coders called "Serpent" has created and published an update to the game client to work with custom servers. This works with the original Lightspeed files, so it is layered as an application with the ID `com.serpentdev.client`.
> - Serpent have also created an optional UI mod called "NewUI". This has the application ID `com.serpentdev.newui`.
> - Another separate group called "OpenShips" are the team running a public server called "Liberation". Their server ID is `net.openships.liberation`. This server uses the Serpent client.

You can see two example manifests in this folder - these are complete versions of the snippets below. It is assumed that the user would have the "SerpentDev" one installed by default, and then add the "OpenShips" one on top.

## The XML file
To avoid confusion with other manifests or XML files, it is recommended (but not required) that the XML file hosted by your server uses the `.sunrise.xml` extension. For example, the manifest for OpenShips may be accessible from https://openships.net/repo/openships.sunrise.xml.

The core of the XML file looks like this:

```xml
<?xml version="1.0" encoding="UTF-8" ?>

<sunrise-manifest version="1.0" game="com.example.starships">
    <name>OpenShips Servers</name>

    <servers>
        <!-- Server tags go here -->
    </servers>

    <applications>
        <!-- Application tags go here -->
    </applications>

    <runtimes>
        <!-- Runtime tags go here -->
    </runtimes>
</sunrise-manifest>
```

On the root **sunrise-manifest** block, the version should always equal this specification's version number. The **game** attribute should be the same across all projects that target the same game - this way the launcher knows to only accept manifests for "game 1" and not "game 2". It is *not* for your server/project name, just the game it targets.

Inside the root element, only the `name` tag is required, which will show in the Sunrise options window as a label for end users to differentiate between what each manifest offers.

The `servers`, `applications`, and `runtimes` blocks are all optional, so you only have to specify what your project actually provides. Most projects for example won't need to specify a runtime.

## Runtimes
Runtimes are intended to provide only the base game as it was published. This system was created so Sunrise could support the case of servers that are based on different versions/expansions of the original game.

Each runtime is installed in a separate folder next to the Sunrise launcher (e.g. `C:\Sunrise\runtimes\com.example.starships.liberation`), and applications are then layered on top of that.

An example runtime would look something like this:

```xml
<runtimes>
    <runtime id="com.example.starships.liberation">
        <name>Liberation</name>
        <publisher>ExampleCorp</publisher>
        <icon>https://example.com/img/starships-icon.png</icon>

        <files>
            <file name="data/ss1.bin" size="350824" md5="48f17b33fcff1cd05001c48f23af01d8" sha512="7fc53d68471e530aab3404bb61751b70135225ee7565420516655943062ff0eb1478e9cd3bf228289f3d5a861725b78162afdee1621374b02b8768f073d5667a">
                <url>http://example.com/repo/starships/data/ss1.bin</url>
            </file>

            <!-- Add more files here... -->
        </files>
    </runtime>
</runtimes>
```

The runtime tag has an `id` attribute.

### Tags supported:
 - `name`: *Required.* Name of the runtime. Keep it simple. Avoid including your game's name, we all know what it's for - just the patch/expansion level it's for.
 - `publisher`: *Required.* Publisher of the runtime.
 - `icon`: *Recommended.* URL to an logo representing the project. Must be square, no larger than 512x512, and as a PNG or JPEG.
 - `files`: *Required.* A list of **file** tags (see below).

### File tags
File tags support the following attributes:

 - `name`: *Required.* Final installed filepath.
 - `size`: *Required.* File size.
 - `md5`: *Required.* MD5 sum of the file.
 - `sha512`: *Recommended.* SHA-512 sum of the file.

Within the file tags is a list of publicly accessible URLs, wrapped in `url` tags, that point to valid mirrors. Other download capabilities other than HTTP/HTTPS may be added in the future.

## Applications
Applications are what most projects will need. This is a custom set of files applied on top of the runtime itself, for servers with custom content or fixes to their game client.

*Please avoid overwriting any other application's files! Try to use unique names for executables and throw any other resources into a new folder, ideally both named after your project. If you're just running a server that uses an existing application's code/resources, don't create a new application but use the Servers functionality instead!*

An example application would look something like this:

```xml
<applications>
    <application id="com.serpentdev.client" type="client" runtime="com.example.starships.liberation" standalone="true">
        <name>My Game</name>
        <publisher>Serpent</publisher>
        <version>v2.1.0</version>
        <icon>https://serpentdev.com/img/icon.png</icon>

        <website type="home">https://serpentdev.com</website>
        <website type="issues">https://git.serpentdev.com/serpent/client/issues</website>
        <news url="http://serpentdev.com/starships.rss" />

        <launcher exec="serpent.exe" params="-patchdir serpent -console" />

        <files>
            <file name="serpent.exe" size="9107968" md5="48f17b33fcff1cd05001c48f23af01d8" sha512="7fc53d68471e530aab3404bb61751b70135225ee7565420516655943062ff0eb1478e9cd3bf228289f3d5a861725b78162afdee1621374b02b8768f073d5667a">
                <url>http://serpentdev.com/repo/starships/serpent.exe</url>
                <url>http://mymirror.com/starships/serpent.exe</url>
            </file>

            <!-- Add more files here... -->
        </files>
    </application>

    <application id="com.serpentdev.newui" type="mod" runtime="com.example.starships.liberation">
        <name>NewUI</name>
        <publisher>Serpent</publisher>
        <version>v0.7.1</version>
        <icon>https://serpentdev.com/img/mymod-icon.png</icon>

        <website type="home">https://serpentdev.com/newui/</website>
        <news url="http://serpentdev.com/newui.rss" />

        <files>
            <file name="images/ui_button.png" size="9107968" md5="48f17b33fcff1cd05001c48f23af01d8" sha512="7fc53d68471e530aab3404bb61751b70135225ee7565420516655943062ff0eb1478e9cd3bf228289f3d5a861725b78162afdee1621374b02b8768f073d5667a">
                <url>http://serpentdev.com/repo/newui/ui_button.png</url>
                <url>http://mymirror.com/newui/ui_button.png</url>
            </file>

            <!-- Add more files here... -->
        </files>
    </application>
</applications>
```

### Attributes supported
 - `id`: *Required.* Unique application ID.
 - `runtime`: *Required.* Target runtime this app depends on.
 - `type`: *Optional.* Defaults to **"client"**. Should either be:
   - `client`: Applications that rely on custom servers to launch the game. Are given the most prominence in the Sunrise UI by way of server tags hooking into them.
   - `mod`: Modifications for the game that don't rely on custom servers. Including a launchable executable file is optional. Examples include "offline modes" of the game *(for including an executable)*, or visual & UI mods *(no executable)*.
   - `tool`: Applications that are standalone and not part of the game, but launching/updating is provided as a mere convenience by Sunrise. Examples include build planning tools, chat parsers etc.
 - `standalone`: *Optional.* Defaults to **"false"**. If false, install this application in the runtime directory. If true, install this application in its own directory.

### Tags supported
 - `name`, `publisher`, and `files` are the same as the Runtimes section. All required.
 - `version`: String containing the version number. There's no set format here as it's not currently used internally, it's just user-facing *(although can optionally be used as a variable - see below)*.
 - `icon` is the same as the Runtimes section, and is recommended but not required.
 - `website`: *Optional.* URL of a website representing the project. There can be multiple website fields, but only one of each type. The valid types are:
   - *home* (main homepage)
   - *forums* (forums)
   - *wiki* (wiki)
   - *changelog* (for patch notes)
   - *support* (for help/support)
   - *issues* (for bug tracking/reporting)
 - `news`: *Optional.* RSS feed containing latest news posts. The launcher will fetch up to 5 latest posts.
 - `launcher`: *Required for types client & tool, optional for type mod.* Name of the EXE file to run in the **exec** attribute, and launch parameters in the **params** attribute. These can also take certain variables (see below). ***DO NOT INCLUDE ANY SERVER DETAILS IN THE PARAMATERS, THESE ARE HANDLED BY THE SERVER TAGS BELOW.***

### Launcher parameter variables
The `<launcher params="" />` value can take several variables, to make updating them less of a pain. These are:

 - `$APP_ID`: App ID
 - `$APP_VERSION`: App version *(if `version` tag is set)*
 - `$RT_ID`: Runtime ID
 - `$APP_PATH` & `$RT_PATH`: App/runtime relative folder path
 - `$APP_ABSPATH` & `$RT_ABSPATH`: App/runtime absolute folder path

## Servers
*This section is not to be confused with the Server/Uptime spec, although the two may merge/become interoperable in the distant future.*

The servers block is filled with multiple `server` tags, to make providing information on your project's main game servers easier.

Users will also be able to define their own custom servers in the Options section of the launcher, but it is recommended to use a server field in your manifest where possible, so that the launcher can automatically update any changes to how players should connect.

```xml
<servers>
    <server id="net.openships.liberation" application="com.serpentdev.client">
        <name>OpenShips Liberation</name>
        <icon>https://openships.net/img/liberation-icon.png</icon>

        <website type="home">https://openships.net</website>
        <website type="forums">https://openships.net/forums/</website>
        <news url="http://openships.net/news.rss" />

        <launcher params="-auth 42.0.66.1" />
    </server>
</servers>
```

The opening tag supports the **id** attribute and an **application** attribute - the latter simply specifies the ID of the application this server is designed to work with.

### Tags supported
 - `name`, is the same as the Applications section, and is required.
 - `icon` and `website` are the same as the Applications section, but are recommended, not required.
 - `launcher`: Takes only the **params** attribute, which is appended to the executable/params of the target application's launcher. This is only really for configuring what IP/URL to connect to.
