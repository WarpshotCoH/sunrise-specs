# Sunrise XML specifications
This repository details the specifications relating to files compatible with the [Sunrise launcher](https://github.com/WarpshotCoH/sunrise-launcher).

There are currently two specifications detailed below. Comments can be sent to Warpshot on the "SCOTS" server.

> ***A note about application/runtime/server IDs***
>
> Project IDs use a naming system called [reverse domain name notation](https://en.wikipedia.org/wiki/Reverse_domain_name_notation), to avoid any conflicts. The basic idea is that you take a domain name owned by the project, printed in reverse, with the project name as the final part of the string. This string should be **all lowercase**, and once a project is published, a project ID **should never be changed** (that's why there's a separate "name" field!).
>
> For example, Google Maps may have `com.google.maps` as their project ID, or GNOME Software as `com.gnome.software`.
>
> If you don't own a domain and are just working off GitHub (or a similar service), use the GitHub Pages domain (**not** their main website domain). For example, the user `Statesman04` may have `io.github.statesman04.myproject` as their project ID.

## Manifest spec
***Current version: 1.0 (working draft)***

Designed as a complete overhaul of the Tequila/Island Rum format.

***See the `Manifest 1.0` folder for more information and examples.***

## Uptime/server spec
***Current version: 1.0 (working draft)***

An simple standard format to return information about server uptime for a project/group of servers in an XML file.

Not to be confused with the `servers` section of the Manifest spec, although the two may merge/become interoperable in the distant future.

***See the `Uptime 1.0` folder for more information and examples.***
