Quinstance 0.1.0
================

A wrapper around VMFInstanceInserter, enabling the use of func_instances in
Quake maps.

Requires a .map in Valve 220 format, and one or more FGDs describing the
entities found therein. Additionally, use in Windows will need version 4 or
later of the .NET Framework, while Linux and OS X usage depends on an
installation of Mono: http://www.mono-project.com

My thanks to James King, "Metapyziks", for both the development of VMFII in the
first place and for the flexible license terms that let me redistribute a copy
along with Quinstance. Learn more about James' project at its Github repository:
https://github.com/Metapyziks/VMFInstanceInserter

http://www.gyroshot.com
robert.martens@gmail.com
@ItEndsWithTens


Usage
-----

    quinstance.exe input [output] -d FGD [-c] [-k] [-t TMPDIR]

    Linux/OS X users will need to add 'mono ' to the head of their command line.

    Parameters:

      input
        The input file to be processed. Must be a Quake .map in Valve 220 format.

      output [optional]
        The file to output after processing. Defaults to input.temp.map.

      -d, --fgd
        Specify one or more FGD files, as a comma separated string, to be
        preprocessed and passed along to VMFII.

      -c, --cleanup [optional]
        Deletes 'output' and renames the associated BSP, PRT, LIN and PTS files.

      -k, --keep [optional]
        Keep the generated temporary files instead of deleting them.

      -t, --tmpdir [optional]
        Specify the directory in which to store temporary files. Defaults to the
        user's temp directory.


Background
----------

The func_instance entity, as seen in the Source engine, is a type of prefab
widget. Build reusable components in one map file, then place instances of
them in others, with changes automatically propagated at compile time.

Uses for the entity range from things like duplicating common architectural
elements or complex entity configurations, to building structures on grid but
positioning them off grid while maintaining ease of editing (and that without
fear of slowly accumulating vertex creep), to having multiple mappers working on
a project concurrently. Instances aren't the solution to every mapping problem,
and have their own quirks, but when you need them they make life a lot easier.


Using instances
---------------

First, you'll need to make the func_instance entity available in your editor.

Included in the 'extra' directory is an FGD you can load into Jackhammer, or any
other editor that supports the same FGD format. TrenchBroom is not among them,
but a TB-ready entity definition can be found in func_instance.fgd. Open the
file in a text editor, copy the TrenchBroom section into

    TrenchBroom/Resources/Defs/Quake.fgd

and remove the leading '//' comment tokens. Restart TrenchBroom and you should
have a new entity in the context menu's Create Point Entity|Func section. Please
do note that I haven't extensively tested my entity definition block with
TrenchBroom! Comments and suggestions welcome.

Next is compiler setup.

Editor-specific instructions are beyond the scope of this readme, but the basic
idea is the same everywhere. Add a compile step, just before CSG, that runs the
Quinstance executable as follows:

    quinstance.exe input.map --fgd "fgd1.fgd,fgd2.fgd,..."

The --fgd parameter needs to be a comma separated list of the FGDs your map
depends on, func_instance.fgd excluded (VMFII already knows what instances are
and how to process them).

Then add a second compile step, immediately after BSP, with the following:

    quinstance.exe input.map --cleanup

The instance resolution process produces an intermediate .map file, which is the
one that actually gets compiled. Running Quinstance with --cleanup will delete
this intermediate, then rename the associated .bsp, .prt, .lin, and .pts files
as appropriate. The BSP log will not be renamed, so QBSP executables that append
to existing logs can do so, and old build output won't be lost.

Finally is actually using the entities.

Place a func_instance in your map at whatever position and orientation you'd
like. Set the entity's Filename key to the relative path of another map,
containing geometry and/or entities, and see that after being run through
Quinstance and compiled, a copy of the other map's contents will have appeared
where the func_instance once was.


func_instance
-------------

key - Display Name - type, default
  Description.

targetname - Fixup Name - target_source, default AutoInstanceX
  A name that will, depending on the fixup style, be prepended or appended to
  any entities. If a Fixup Style is set, but a Fixup Name is not provided, an
  automatically generated name will be used. Keep in mind that even with fixup
  enabled and a name set, you can selectively avoid fixup by giving entities
  names starting with @.

file - Filename - string, default ""
  A path, relative to the current map file's location, pointing to the map you'd
  like to copy in.

fixup_style - Fixup Style - integer, default 0
  The method by which entity names will be fixed up.
  0 : Prefix
  1 : Postfix
  2 : None

replace0X - Replace - string, default ""
  A replacement parameter that takes the form of $variable value. For example,
  set this field to $brightness 750 and any occurrence of $brightness inside the
  Filename map will be replaced with 750 when the instances are collapsed.
  Materials can also be replaced, with #. For example, setting a replacement
  variable to #SKY1 DOPEFISH will retexture any surfaces in the Filename map,
  replacing the classic purple sky with everyone's favorite goofy fish.
  If you need more than ten replacements, don't forget you can turn off
  SmartEdit (if applicable) and add keys manually: replace11, replace12, etc.

For more information about func_instances as they're implemented in the Source
engine, see https://developer.valvesoftware.com/wiki/Func_instance


Changes
-------

0.1.0 - October 19th, 2015
  Initial release