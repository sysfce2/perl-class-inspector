# Class::Inspector ![static](https://github.com/uperl/Class-Inspector/workflows/static/badge.svg) ![linux](https://github.com/uperl/Class-Inspector/workflows/linux/badge.svg) ![macos](https://github.com/uperl/Class-Inspector/workflows/macos/badge.svg) ![windows](https://github.com/uperl/Class-Inspector/workflows/windows/badge.svg)

Get information about a class and its structure

# SYNOPSIS

```perl
use Class::Inspector;

# Is a class installed and/or loaded
Class::Inspector->installed( 'Foo::Class' );
Class::Inspector->loaded( 'Foo::Class' );

# Filename related information
Class::Inspector->filename( 'Foo::Class' );
Class::Inspector->resolved_filename( 'Foo::Class' );

# Get subroutine related information
Class::Inspector->functions( 'Foo::Class' );
Class::Inspector->function_refs( 'Foo::Class' );
Class::Inspector->function_exists( 'Foo::Class', 'bar' );
Class::Inspector->methods( 'Foo::Class', 'full', 'public' );

# Find all loaded subclasses or something
Class::Inspector->subclasses( 'Foo::Class' );
```

# DESCRIPTION

Class::Inspector allows you to get information about a loaded class. Most or
all of this information can be found in other ways, but they aren't always
very friendly, and usually involve a relatively high level of Perl wizardry,
or strange and unusual looking code. Class::Inspector attempts to provide
an easier, more friendly interface to this information.

# METHODS

## installed

```perl
my $bool = Class::Inspector->installed($class);
```

The `installed` static method tries to determine if a class is installed
on the machine, or at least available to Perl. It does this by wrapping
around `resolved_filename`.

Returns true if installed/available, false if the class is not installed,
or `undef` if the class name is invalid.

## loaded

```perl
my $bool = Class::Inspector->loaded($class);
```

The `loaded` static method tries to determine if a class is loaded by
looking for symbol table entries.

This method it uses to determine this will work even if the class does not
have its own file, but is contained inside a single file with multiple
classes in it. Even in the case of some sort of run-time loading class
being used, these typically leave some trace in the symbol table, so an
[Autoload](https://metacpan.org/pod/Autoload) or [Class::Autouse](https://metacpan.org/pod/Class::Autouse)-based class should correctly appear
loaded.

Returns true if the class is loaded, false if not, or `undef` if the
class name is invalid.

## filename

```perl
my $filename = Class::Inspector->filename($class);
```

For a given class, returns the base filename for the class. This will NOT
be a fully resolved filename, just the part of the filename BELOW the
`@INC` entry.

```
print Class->filename( 'Foo::Bar' );
> Foo/Bar.pm
```

This filename will be returned with the right separator for the local
platform, and should work on all platforms.

Returns the filename on success or `undef` if the class name is invalid.

## resolved\_filename

```perl
my $filename = Class::Inspector->resolved_filename($class);
my $filename = Class::Inspector->resolved_filename($class, @try_first);
```

For a given class, the `resolved_filename` static method returns the fully
resolved filename for a class. That is, the file that the class would be
loaded from.

This is not necessarily the file that the class WAS loaded from, as the
value returned is determined each time it runs, and the `@INC` include
path may change.

To get the actual file for a loaded class, see the `loaded_filename`
method.

Returns the filename for the class, or `undef` if the class name is
invalid.

## loaded\_filename

```perl
my $filename = Class::Inspector->loaded_filename($class);
```

For a given loaded class, the `loaded_filename` static method determines
(via the `%INC` hash) the name of the file that it was originally loaded
from.

Returns a resolved file path, or false if the class did not have it's own
file.

## functions

```perl
my $arrayref = Class::Inspector->functions($class);
```

For a loaded class, the `functions` static method returns a list of the
names of all the functions in the classes immediate namespace.

Note that this is not the METHODS of the class, just the functions.

Returns a reference to an array of the function names on success, or `undef`
if the class name is invalid or the class is not loaded.

## function\_refs

```perl
my $arrayref = Class::Inspector->function_refs($class);
```

For a loaded class, the `function_refs` static method returns references to
all the functions in the classes immediate namespace.

Note that this is not the METHODS of the class, just the functions.

Returns a reference to an array of `CODE` refs of the functions on
success, or `undef` if the class is not loaded.

## function\_exists

```perl
my $bool = Class::Inspector->function_exists($class, $functon);
```

Given a class and function name the `function_exists` static method will
check to see if the function exists in the class.

Note that this is as a function, not as a method. To see if a method
exists for a class, use the `can` method for any class or object.

Returns true if the function exists, false if not, or `undef` if the
class or function name are invalid, or the class is not loaded.

## methods

```perl
my $arrayref = Class::Inspector->methods($class, @options);
```

For a given class name, the `methods` static method will returns ALL
the methods available to that class. This includes all methods available
from every class up the class' `@ISA` tree.

Returns a reference to an array of the names of all the available methods
on success, or `undef` if the class name is invalid or the class is not
loaded.

A number of options are available to the `methods` method that will alter
the results returned. These should be listed after the class name, in any
order.

```perl
# Only get public methods
my $method = Class::Inspector->methods( 'My::Class', 'public' );
```

- public

    The `public` option will return only 'public' methods, as defined by the Perl
    convention of prepending an underscore to any 'private' methods. The `public`
    option will effectively remove any methods that start with an underscore.

- private

    The `private` options will return only 'private' methods, as defined by the
    Perl convention of prepending an underscore to an private methods. The
    `private` option will effectively remove an method that do not start with an
    underscore.

    **Note: The `public` and `private` options are mutually exclusive**

- full

    `methods` normally returns just the method name. Supplying the `full` option
    will cause the methods to be returned as the full names. That is, instead of
    returning `[ 'method1', 'method2', 'method3' ]`, you would instead get
    `[ 'Class::method1', 'AnotherClass::method2', 'Class::method3' ]`.

- expanded

    The `expanded` option will cause a lot more information about method to be
    returned. Instead of just the method name, you will instead get an array
    reference containing the method name as a single combined name, a la `full`,
    the separate class and method, and a CODE ref to the actual function ( if
    available ). Please note that the function reference is not guaranteed to
    be available. `Class::Inspector` is intended at some later time, to work
    with modules that have some kind of common run-time loader in place ( e.g
    `Autoloader` or `Class::Autouse` for example.

    The response from `methods( 'Class', 'expanded' )` would look something like
    the following.

    ```
    [
      [ 'Class::method1',   'Class',   'method1', \&Class::method1   ],
      [ 'Another::method2', 'Another', 'method2', \&Another::method2 ],
      [ 'Foo::bar',         'Foo',     'bar',     \&Foo::bar         ],
    ]
    ```

## subclasses

```perl
my $arrayref = Class::Inspector->subclasses($class);
```

The `subclasses` static method will search then entire namespace (and thus
**all** currently loaded classes) to find all classes that are subclasses
of the class provided as a the parameter.

The actual test will be done by calling `isa` on the class as a static
method. (i.e. `My::Class->isa($class)`.

Returns a reference to a list of the loaded classes that match the class
provided, or false is none match, or `undef` if the class name provided
is invalid.

# SEE ALSO

[Class::Handle](https://metacpan.org/pod/Class::Handle), [Class::Inspector::Functions](https://metacpan.org/pod/Class::Inspector::Functions)

# AUTHOR

Original author: Adam Kennedy <adamk@cpan.org>

Current maintainer: Graham Ollis <plicease@cpan.org>

Contributors:

Tom Wyant

Steffen Müller

Kivanc Yazan (KYZN)

# COPYRIGHT AND LICENSE

This software is copyright (c) 2002-2024 by Adam Kennedy.

This is free software; you can redistribute it and/or modify it under
the same terms as the Perl 5 programming language system itself.
