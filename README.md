[![Build Status](https://drone.io/github.com/ical4j/ical4j/status.png)](https://drone.io/github.com/ical4j/ical4j/latest) [![Gitter](https://badges.gitter.im/Join Chat.svg)](https://gitter.im/ical4j/ical4j?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

# iCal4j - Release Notes

 - For a concise description of the goals and directions of iCal4j please
 take a look at the [open issues](https://github.com/ical4j/ical4j/issues).

 - You will find examples of how to use iCal4j in [the wiki](https://github.com/ical4j/ical4j/wiki)
 and throughout the [API documentation](https://ical4j.github.io/docs/ical4j/api).

 - Detailed descriptions of changes included in each release may be found
 in the [CHANGELOG](https://ical4j.github.io/docs/ical4j/release-notes).
 
 - iCal4j was created with the help of [Open Source](http://opensource.org) software.


## System Requirements

 - Java 6 or later
 
See [here](https://github.com/ical4j/ical4j/docs/Dependencies.md) for further details.

## Building with Gradle

iCal4j includes the Gradle wrapper for a simpler and more consistent build.

**Run unit tests**

    ./gradlew clean test

**Build a new release**

    ./gradlew clean test release -Prelease.forceVersion=2.0.0

**Upload release binaries and packages**

    RELEASE_VERSION=2.0.0 ./gradlew uploadArchives uploadDist


 
## Relaxed Parsing

 iCal4j now has the capability to "relax" its parsing rules to enable parsing of
 *.ics files that don't properly conform to the iCalendar specification (RFC2445)
 
 - You can relax iCal4j's unfolding rules by specifying the following system property:
 
       ical4j.unfolding.relaxed=true
 
 Note that I believe this problem is not restricted to Mozilla calendaring
 products, but rather may be caused by UNIX/Linux-based applications relying on the
 default newline character (LF) to fold long lines (KOrganizer also seems to have this
 problem). This is, however, still incorrect as by definition long lines are folded
 using a (CRLF) combination.
 
 I've obtained a couple of samples of non-standard iCalendar files that I've included
 in the latest release (0.9.11). There is a Sunbird, phpicalendar, and a KOrganizer
 sample there (open them in Notepad on Windows to see what I mean).

 It seems that phpicalendar and KOrganizer always use LF instead of CRLF, and in
 addition KOrganizer seems to fold all property parameters and values (similar to
 Mozilla Calendar/Sunbird).

 Mozilla Calendar/Sunbird uses CRLF to fold all property parameter/values, however it
 uses just LF to fold long lines (i.e. longer than 75 characters).

 The latest release of iCal4j includes changes to UnfoldingReader that should work
 correctly with Mozilla Calendar/Sunbird, as long as the ical4j.unfolding.relaxed
 system property is set to true.

 KOrganizer/phpicalendar files should also work with the relaxed property, although
 because ALL lines are separated with just LF it also relies on the StreamTokenizer to
 correctly identify LF as a newline on Windows, and CRLF as a newline on UNIX/Linux. The
 API documentation for Java 1.5 says that it does do this, so if you still see problems
 with parsing it could be a bug in the Java implementation.
 
 
 - You may also relax the parsing rules of iCal4j by setting the following system property:
 
       ical4j.parsing.relaxed=true
 
 This property is intended as a general relaxation of parsing rules to allow for parsing
 otherwise invalid calendar files. Initially enabling this property will allow for the
 creation of properties and components with illegal names (e.g. Mozilla Calendar's "X"
 property). Note that although this will allow for parsing calendars with illegal names,
 validation will still identify such names as an error in the calendar model.


 - Microsoft Outlook also appears to provide quoted TZID parameter values, as follows:
 
       DTSTART;TZID="Pacific Time (US & Canada),Tijuana":20041011T223000
 
 To allow for compatibility with such iCalendar files you should specify the
 following system property:
 
     ical4j.compatibility.outlook=true
 
 The full set of system properties may be found in
 net.fortuna.ical4j.util.CompatibilityHints.


## iCal4j and Timezones

 Supporting timezones in an iCalendar implementation can be a complicated process,
 mostly due to the fact that there is not a definitive list of timezone definitions
 used by all iCalendar implementations. This means that an iCalendar file may be
 produced by one implementation and, if the file does not include all definitions
 for timezones relevant to the calendar properties, an alternate implementation
 may not know how to interpret the timezone identified in the calendar (or worse,
 it may interpret the timezone differently to the original implementation). All
 of these possibilities mean unpredictable behaviour which, to put it nicely, is
 not desireable.
 
 iCal4j approaches the problem of timezones in two ways: The first and by far the
 preferred approach is for iCalendar files to include definitions for all timezones
 referenced in the calendar object. To support this, when an existing calendar is
 parsed a list of VTimeZone definitions contained in the calendar is constructed.
 This list may then be queried whenever a VTimeZone definition is required.
 
 The second approach is to rely on a registry of VTimeZone definitions. iCal4j
 includes a default registry of timezone definitions (derived from the Olson timezone
 database - a defacto standard for timezone definitions), or you may also provide your
 own registry implementation from which to retreieve timezones. This approach is
 required when constructing new iCalendar files.
 
 Note that the intention of the iCal4j model is not to provide continuous validation
 feedback for every change in the model. For this reason you are free to change
 timezones on Time objects, remove or add TzId parameters, remove or add VTimeZone
 definitions, etc. without restriction. However when validation is run (automatically
 on output of the calendar) you will be notified if the changes are invalid.
 


## Redistribution:

If you intend to use and distribute iCal4j in your own project please
follow these very simple guidelines:
 
 - Make a copy of the LICENSE, rename it to LICENSE.ical4j, and save
 it to the directory where you are re-distributing the iCal4j JAR.
 
 - I don't recommend extracting the iCal4j classes from its JAR and package
 in another JAR along with other classes. It may lead to version incompatibilites
 in the future. Rather I would suggest to include the ical4j.jar in your classpath
 as required.
