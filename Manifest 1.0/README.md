# Sunrise manifest specification
***Version 1.0 (working draft)***

The Sunrise manifest format is partially based on the previous Tequila/Island Rum format, but with a much cleaner separation of projects from both each other and the base game. This is done by splitting projects into "applications" and "runtimes", partially inspired by the Flatpak and Snap packaging systems on Linux (though much less complex). Servers are also split into a separate area, to allow easier distribution and access to custom servers.

## The XML file
The core of the `manifest.xml` file looks like this:

```xml
<?xml version="1.0" encoding="UTF-8" ?>

<sunrise-manifest version="1.0" game="com.example.game1">
    <name>My First Manifest</name>

    <servers>
        <!-- Server blocks go here -->
    </servers>

    <applications>
        <!-- Application blocks go here -->
    </applications>

    <runtimes>
        <!-- Runtime blocks go here -->
    </runtimes>
</sunrise-manifest>
```

On the main **sunrise-manifest** block, the version should always equal the specification's version number. The **game** attribute should be the same across all projects that target the same game - this way the launcher knows to only accept manifests for "game 1" and not "game 2".

Inside the main block, only the **name** tag is required, which will show in the Sunrise options window as a label for end users to differentiate between what each manifest offers.

The **servers**, **applications**, and **runtimes** blocks are all optional, so you only have to specify what your project actually provides. Most projects for example won't need to specify a runtime.

## Runtimes
Runtimes are intended to provide only the base game as it was published. This system was created so Sunrise could support projects in the case that servers are able to run alternate issues in the future (e.g. a new server offering Issue 1, or Issue 23).

Each runtime is installed in a separate folder next to the Sunrise launcher, and applications are then layered on top of that.

An example runtime would look something like this:

```xml
<runtimes>
    <runtime id="com.example.myruntime">
        <name>Runtime Issue 1</name>
        <publisher>Lorem Ipsum</publisher>

        <files>
            <file name="qt4.dll" size="350824" md5="7652657fa392fb9418ccf309145dd7b4" sha512="5814080fe36fa332950c1e56debbc0da7afee699708d8d4af0e4310ed27ce37124b5cf80a08effc9bd4ee5806bcc0a9f7ea6bdc64cee7ef5057fc20d39c9d182">
                <url>http://example.com/repo/myruntime/qt4.dll</url>
            </file>

            <!-- Add more files here... -->
        </files>
    </runtime>
</runtimes>
```

Fields supported:
 - `name`: *Required.* Name of the runtime. Keep it simple. Avoid including your game's name, we all know what it's for - just the patch/expansion level it's for.
 - `publisher`: *Required.* Publisher of the runtime.
 - `files`: *Required.* A list of **file** tags (see below).

### File tags
File tags support:

 - `name`: *Required.* Final installed filepath.
 - `size`: *Required.* File size.
 - `md5`: *Required.* MD5 sum of the file.
 - `sha512`: *Recommended.* SHA-512 sum of the file.

Within the file tags is a list of publicly accessible URLs that point to valid mirrors. Other download capabilities other than HTTP/HTTPS may be added in the future.

## Applications
Applications are what most projects will only need. This is a custom set of files.

*Please avoid overwriting any other application's files, try to use unique names for executables and throw any other resources into a new folder, ideally both named after your project. If you're just running a server that uses an existing application's code/resources, don't create a new application but use the Servers functionality instead!*

An example application would look something like this:

```xml
<applications>
    <application id="com.example.mygame" type="client" runtime="com.example.myruntime">
        <name>My Game</name>
        <publisher>Lorem Ipsum</publisher>
        <icon>https://example.com/img/mygame-icon.png</icon>

        <website type="home">https://example.com/mygame/</website>
        <website type="forums">https://example.com/forums/</website>

        <launcher exec="game.exe" params="-patchdir mygame -noversioncheck" />

        <news url="http://example.com/mygame.rss" />

        <files>
            <file name="game.exe" size="9107968" md5="41616f6dc3501bbb1c9b2bac0b51099e" sha512="d7ce630c91a4bd10499ae5b88c7116f773527d52cd80a0b82544a3dd7a39692afc76ae61a9f203f86020fa615f6c8e17e066b900314caa051bb3cde6d5f261d6">
                <url>http://example.com/repo/mygame/game.exe</url>
            </file>

            <!-- Add more files here... -->
        </files>
    </application>

    <application id="com.example.mod" type="mod" runtime="com.example.myruntime">
        <name>My UI Mod</name>
        <publisher>Lorem Ipsum</publisher>
        <icon>https://example.com/img/mymod-icon.png</icon>

        <website type="home">https://example.com/mymod/</website>
        <website type="forums">https://example.com/forums/</website>

        <news url="http://example.com/mymod.rss" />

        <files>
            <file name="images/ui_button.png" size="9107968" md5="41616f6dc3501bbb1c9b2bac0b51099e" sha512="d7ce630c91a4bd10499ae5b88c7116f773527d52cd80a0b82544a3dd7a39692afc76ae61a9f203f86020fa615f6c8e17e066b900314caa051bb3cde6d5f261d6">
                <url>http://example.com/repo/mymod/ui_button.png</url>
            </file>

            <!-- Add more files here... -->
        </files>
    </application>
</applications>
```

Notice in the opening tag, there is a **runtime** attribute and a **type** attribute. The **runtime** attribute simply specifies the ID of the runtime folder this application should be installed to.

The **type** attribute should either be:
 - `client`: Applications that rely on custom servers to launch the game. Are given the most prominence in the Sunrise UI by way of server blocks hooking into them.
 - `mod`: Modifications for the game that don't rely on custom servers. Including a launchable executable file is optional. Examples include "offline modes" of the game *(for including an executable)*, or visual & UI mods *(no executable)*.
 - `tool`: Applications that don't rely on game data at all and are standalone, but with launching/updating provided as a mere convenience by Sunrise. Examples include build planning tools. These are installed separate

### Fields supported
 - `name`, `publisher`, and `files` are the same as the Runtimes section. All required.
 - `launcher`: *Required for types client & tool, optional for type mod.* Name of the EXE file to run in the **exec** attribute, and launch parameters in the **params** attribute. ***DO NOT INCLUDE YOUR SERVER DETAILS IN THE PARAMATERS, THESE ARE HANDLED BY THE SERVER BLOCKS.***
 - `icon`: *Recommended.* URL to an logo representing the project. Must be square, no larger than 512x512, and as a PNG or JPEG.
 - `website`: *Optional.* URL of a website representing the project. There can be multiple website fields, but only one of each type. The valid types are:
   - *home* (main homepage)
   - *forums* (forums)
   - *wiki* (wiki)
   - *changelog* (for patch notes)
   - *support* (for help/support)
   - *issues* (for bug tracking/reporting)
 - `news`: *Optional.* RSS feed containing latest news posts. The launcher will fetch up to 5 latest posts.

## Servers
***TO BE REWRITTEN SHORTLY, IGNORE BELOW***

*Not to be confused with the Server/Uptime spec, although the two may merge/become interoperable in the distant future.*

The servers block is filled with multiple `server` fields, to make providing information on your project's main game servers easier.

Users will also be able to define their own custom servers in the Options section of the launcher, but it is recommended to use a server field in your manifest where possible, so that the launcher can automatically update any changes to how players should connect.

```xml
<servers>
    <server id="com.example.myserver" application="com.example.mygame">
        <name>Test Server</name>
        <publisher>Server Group</publisher>
        <icon>https://example.com/img/myserver-icon.png</icon>

        <website type="home">https://example.com/myserver/</website>
        <website type="forums">https://example.com/forums/</website>

        <launcher params="-auth 42.0.66.1" />

        <news url="http://example.com/myserver.rss" />
    </server>
</servers>
```


Notice in the opening tag, there is a **runtime** attribute and a **custom-servers** attribute. The **runtime** attribute simply specifies the ID of the runtime folder this application should be installed to.

### Tags supported
 - `id`: Unique ID.
 - `name`: Name of the project/server, to be listed in a dropdown on the launcher.
 - `application`: The ID of the application that this server is for.
 - `params`: Launch parameters to add to the game, that specify how to access the server.
