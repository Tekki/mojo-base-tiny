# NAME

Mojo::Base::Tiny - Minimal base class for !Mojo projects

# SYNOPSIS

    package Cat;
    use Mojo::Base::Tiny -base;

    has name => 'Nyan';
    has ['age', 'weight'] => 4;

    package Tiger;
    use Mojo::Base::Tiny 'Cat';

    has friend  => sub { Cat->new };
    has stripes => 42;

    package main;
    use Mojo::Base::Tiny -strict;

    my $mew = Cat->new(name => 'Longcat');
    say $mew->age;
    say $mew->age(3)->weight(5)->age;

    my $rawr = Tiger->new(stripes => 38, weight => 250);
    say $rawr->tap(sub { $_->friend->name('Tacgnol') })->weight;

# DESCRIPTION

[Mojo::Base::Tiny](https://metacpan.org/pod/Mojo::Base::Tiny) is a simple base class for Perl projects with fluent
interfaces.

It is nothing else than [Mojo::Base](https://metacpan.org/pod/Mojo::Base) in a single file without dependencies
outside the core modules (or to be correct, on Perl 5.20 and older you need
[Sub::Util](https://metacpan.org/pod/Sub::Util) 1.41). You can copy it directly to your project in all the
"I can't (or don't want to) install [Mojolicious](https://metacpan.org/pod/Mojolicious)" cases.

    # Automatically enables "strict", "warnings", "utf8" and Perl 5.10 features
    use Mojo::Base::Tiny -strict;
    use Mojo::Base::Tiny -base;
    use Mojo::Base::Tiny 'SomeBaseClass';
    use Mojo::Base::Tiny -role;

All four forms save a lot of typing. Note that role support depends on
[Role::Tiny](https://metacpan.org/pod/Role::Tiny) (2.000001+).

    # use Mojo::Base::Tiny -strict;
    use strict;
    use warnings;
    use utf8;
    use feature ':5.10';
    use mro;
    use IO::Handle ();

    # use Mojo::Base::Tiny -base;
    use strict;
    use warnings;
    use utf8;
    use feature ':5.10';
    use mro;
    use IO::Handle ();
    push @ISA, 'Mojo::Base::Tiny';
    sub has { Mojo::Base::Tiny::attr(__PACKAGE__, @_) }

    # use Mojo::Base::Tiny 'SomeBaseClass';
    use strict;
    use warnings;
    use utf8;
    use feature ':5.10';
    use mro;
    use IO::Handle ();
    require SomeBaseClass;
    push @ISA, 'SomeBaseClass';
    sub has { Mojo::Base::Tiny::attr(__PACKAGE__, @_) }

    # use Mojo::Base::Tiny -role;
    use strict;
    use warnings;
    use utf8;
    use feature ':5.10';
    use mro;
    use IO::Handle ();
    use Role::Tiny;
    sub has { Mojo::Base::Tiny::attr(__PACKAGE__, @_) }

On Perl 5.20+ you can also use the `-signatures` flag with all four forms and
enable support for [subroutine signatures](https://metacpan.org/pod/perlsub#Signatures).

    # Also enable signatures
    use Mojo::Base::Tiny -strict, -signatures;
    use Mojo::Base::Tiny -base, -signatures;
    use Mojo::Base::Tiny 'SomeBaseClass', -signatures;
    use Mojo::Base::Tiny -role, -signatures;

This will also disable experimental warnings on versions of Perl where this
feature was still experimental.

# FLUENT INTERFACES

Fluent interfaces are a way to design object-oriented APIs around method
chaining to create domain-specific languages, with the goal of making the
readablity of the source code close to written prose.

    package Duck;
    use Mojo::Base::Tiny -base;

    has 'name';

    sub quack {
      my $self = shift;
      my $name = $self->name;
      say "$name: Quack!"
    }

[Mojo::Base::Tiny](https://metacpan.org/pod/Mojo::Base::Tiny) will help you with this by having all attribute accessors created
with ["has"](#has) (or ["attr"](#attr)) return their invocant (`$self`) whenever they
are used to assign a new attribute value.

    Duck->new->name('Donald')->quack;

In this case the `name` attribute accessor is called on the object created by
`Duck->new`. It assigns a new attribute value and then returns the `Duck`
object, so the `quack` method can be called on it afterwards. These method
chains can continue until one of the methods called does not return the `Duck`
object.

# FUNCTIONS

[Mojo::Base::Tiny](https://metacpan.org/pod/Mojo::Base::Tiny) implements the following functions, which can be imported with
the `-base` flag or by setting a base class.

## has

    has 'name';
    has ['name1', 'name2', 'name3'];
    has name => 'foo';
    has name => sub {...};
    has ['name1', 'name2', 'name3'] => 'foo';
    has ['name1', 'name2', 'name3'] => sub {...};
    has name => sub {...}, weak => 1;
    has name => undef, weak => 1;
    has ['name1', 'name2', 'name3'] => sub {...}, weak => 1;

Create attributes for hash-based objects, just like the ["attr"](#attr) method.

# METHODS

[Mojo::Base::Tiny](https://metacpan.org/pod/Mojo::Base::Tiny) implements the following methods.

## attr

    $object->attr('name');
    SubClass->attr('name');
    SubClass->attr(['name1', 'name2', 'name3']);
    SubClass->attr(name => 'foo');
    SubClass->attr(name => sub {...});
    SubClass->attr(['name1', 'name2', 'name3'] => 'foo');
    SubClass->attr(['name1', 'name2', 'name3'] => sub {...});
    SubClass->attr(name => sub {...}, weak => 1);
    SubClass->attr(name => undef, weak => 1);
    SubClass->attr(['name1', 'name2', 'name3'] => sub {...}, weak => 1);

Create attribute accessors for hash-based objects, an array reference can be
used to create more than one at a time. Pass an optional second argument to set
a default value, it should be a constant or a callback. The callback will be
executed at accessor read time if there's no set value, and gets passed the
current instance of the object as first argument. Accessors can be chained, that
means they return their invocant when they are called with an argument.

These options are currently available:

- weak

        weak => $bool

    Weaken attribute reference to avoid
    [circular references](https://metacpan.org/pod/perlref#Circular-References) and memory leaks.

## new

    my $object = SubClass->new;
    my $object = SubClass->new(name => 'value');
    my $object = SubClass->new({name => 'value'});

This base class provides a basic constructor for hash-based objects. You can
pass it either a hash or a hash reference with attribute values.

## tap

    $object = $object->tap(sub {...});
    $object = $object->tap('some_method');
    $object = $object->tap('some_method', @args);

Tap into a method chain to perform operations on an object within the chain
(also known as a K combinator or Kestrel). The object will be the first argument
passed to the callback, and is also available as `$_`. The callback's return
value will be ignored; instead, the object (the callback's first argument) will
be the return value. In this way, arbitrary code can be used within (i.e.,
spliced or tapped into) a chained set of object method calls.

    # Longer version
    $object = $object->tap(sub { $_->some_method(@args) });

    # Inject side effects into a method chain
    $object->foo('A')->tap(sub { say $_->foo })->foo('B');

## with\_roles

    my $new_class = SubClass->with_roles('SubClass::Role::One');
    my $new_class = SubClass->with_roles('+One', '+Two');
    $object       = $object->with_roles('+One', '+Two');

Create a new class with one or more [Role::Tiny](https://metacpan.org/pod/Role::Tiny) roles. If called on a class
returns the new class, or if called on an object reblesses the object into the
new class. For roles following the naming scheme `MyClass::Role::RoleName` you
can use the shorthand `+RoleName`. Note that role support depends on
[Role::Tiny](https://metacpan.org/pod/Role::Tiny) (2.000001+).

    # Create a new class with the role "SubClass::Role::Foo" and instantiate it
    my $new_class = SubClass->with_roles('+Foo');
    my $object    = $new_class->new;

# SEE ALSO

[Mojo::Base](https://metacpan.org/pod/Mojo::Base), [Mojolicious](https://metacpan.org/pod/Mojolicious).

# AUTHOR

Sebastian Riedel - `sri@cpan.org`

William Lindley - `wlindley+remove+this@wlindley.com`

Maxim Vuets - `mvuets@cpan.org`

Joel Berger - `joel.a.berger@gmail.com`

Jan Henning Thorsen - `jhthorsen@cpan.org`

Dan Book - `dbook@cpan.org`

Elmar S. Heeb - `heeb@cpan.org`

Dotan Dimet - `dotan@cpan.org`

Zoffix Znet - `cpan@zoffix.com`

Ask Bjørn Hansen - `ask@perl.org`

Tekki (Rolf Stöckli) - `tekki@cpan.org`

Mohammad S Anwar - `mohammad.anwar@yahoo.com`

# COPYRIGHT

© 2008-2019 Sebastian Riedel and others.

© 2019 Tekki (Rolf Stöckli).

This program is free software, you can redistribute it and/or modify it under the terms of the Artistic License version 2.0.
