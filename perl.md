---
layout: default
title: Perl DaZeus documentation
---

# NAME

DaZeus - Perl interface to the DaZeus 2 Socket API

# SYNPOSIS

    use DaZeus;
    my $dazeus = DaZeus->connect("unix:/tmp/dazeus.sock");
    # or:
    my $dazeus = DaZeus->connect("tcp:localhost:1234");
    

    # Get connection status
    my $networks = $dazeus->networks();
    foreach (@$networks) {
      my $channels = $dazeus->channels($n);
      print "$_:\n";
      foreach (@$channels) {
        print "  $_\n";
      }
    }
    

    $dazeus->subscribe("JOINED", sub { warn "JOINED event received!" });
    

    $dazeus->subscribe(qw/PRIVMSG NOTICE/);
    while(my $event = $dazeus->handleEvent()) {
      next if($event->{'event'} eq "JOINED");
      my ($network, $sender, $channel, $message)
        = @{$event->{'params'}};
      print "[$network $channel] <$sender> $message\n";
      my $destination = $channel eq "msg" ? $sender : $channel;
      $dazeus->message($network, $destination, $message);
    }

# DESCRIPTION

This module provides a Perl interface to the DaZeus 2 Socket API, so a Perl
application can act as a DaZeus 2 plugin. The module supports receiving events
and sending messages. See also the "examples" directory that comes with the
Perl bindings.

# METHODS

## connect($socket)

Creates a DaZeus object connected to the given socket. Returns the object if
the initial connection succeeded; otherwise, calls die(). If, after this
initial connection, the connection fails, for example because the bot is
restarted, the module will re-connect.

The given socket name must either start with "tcp:" or "unix:"; in the case of
a UNIX socket it must contain a path to the UNIX socket, in the case of TCP it
may end in a port number. IPv6 addresses must have a port number attached, so
it can be distinguished from the rest of the address; in "tcp:2001:db8::1:1234"
1234 is the port number.

If you give an object through $socket, the module will attempt to use it
directly as the target socket.

## socket()

Returns the internal UNIX or TCP socket used for communication. This call is
useful if you want to watch multiple sockets, for example, using the select()
call. Every time the socket can be read, you can call the handleEvents() method
to process any incoming events. Do not call read() or write() on this socket,
or other calls that change internal socket state.

If the given DaZeus object was not connected, a new connection is opened and
a valid socket will still be returned.

## networks()

Returns a list of active networks on this DaZeus instance, or calls
die() if communication failed.

## channels($network)

Returns a list of joined channels on the given network, or calls
die() if communication failed.

## message($network, $channel, $message)

Sends given message to given channel on given network, or calls die()
if communication failed.

## action($network, $channel, $message)

Like message(), but sends the message as a CTCP ACTION (as if "/me" was used).

## sendNames($network, $channel)

Requests a NAMES command being sent for the given channel on the given network.
After this, a NAMES event will be produced using the normal event system
described below, if the IRC server behaves correctly. Calls die() if
communication failed.

## sendWhois($network, $nick)

Requests a WHOIS command being sent for the given nick on the given network.
After this, a WHOIS event will be produced using the normal event system
described below, if the IRC server behaves correctly. Calls die() if
communication failed.

## join($network, $channel)

Requests a JOIN command being sent for the given channel on the given network.
After this, a JOIN event will be produced using the normal event system
described below, if the IRC server behaves correctly and the channel was not
already joined. Calls die() if communication failed.

## part($network, $channel)

Requests a PART command being sent for the given channel on the given network.
After this, a PART event will be produced using the normal event system
described below, if the IRC server behaves correctly and the channel was
joined. Calls die() if communication failed.

## getNick($network)

Requests the current nickname on given network, and returns it. Calls die()
if communication failed.

## getConfig($name)

Retrieves the given variable from the configuration file and returns
its value. Calls die() if communication failed.

## getProperty($name, \[$network, \[$receiver, \[$sender\]\]\])

Retrieves the given variable from the persistent database and returns
its value. Optionally, context can be given for this property request,
so properties stored earlier using a specific scope can be correctly
matched against this request, and the most specific match will be
returned. Calls die() if communication failed.

## setProperty($name, $value, \[$network, \[$receiver, \[$sender\]\]\])

Stores the given variable to the persistent database. Optionally, context can
be given for this property, so multiple properties with the same name and
possibly overlapping context can be stored and later returned. This is useful
in situations where you want different settings per network, but also
overriding settings per channel. Calls die() if communication failed.

## unsetProperty($name, \[$network, \[$receiver, \[$sender\]\]\])

Unsets the given variable with given context from the persistent database. If
no variable was found with the exact given context, no variables are removed.
Calls die() if communication failed.

## getPropertyKeys($name, \[$network, \[$receiver, \[$sender\]\]\])

Retrieves all keys in a given namespace. I.e. if example.foo and example.bar
were stored earlier, and getPropertyKeys("example") is called, "foo" and "bar"
will be returned from this method. Calls die() if communication failed.

## subscribe($event, \[$event, \[$event, ..., \[$coderef\]\]\])

Subscribes to the given events. If the last parameter is a code reference, it
will be called automatically every time one of the given events hits, with the
DaZeus object as the first parameter, and the received event as the second.

## unsubscribe($event, \[$event, \[$event, ...\]\])

Unsubscribe from the given events. They will no longer be received, until
subscribe() is called again.

## handleEvent($timeout)

Returns all events that can be returned as soon as possible, but no longer
than the given $timeout. If $timeout is zero, do not block. If timeout is
undefined, wait forever until the first event arrives.

"As soon as possible" means that if there are cached events already read from
the socket earlier, the socket will not be touched. Otherwise, if events are
available for reading, they will be immediately read. Only if no events were
cached, none were available for reading, and $timeout is not zero will this
function block to retrieve events. (See also handleEvents().)

This method only returns a single event, in order of receiving. For every
returned event, the event handler (if given to subscribe()), is called.

## handleEvents()

Handle and return as much events as possible without blocking. Also calls the
event handler for every returned event.

# AUTHOR

Sjors Gielen, <dazeus@sjorsgielen.nl>

# COPYRIGHT AND LICENSE

Copyright (C) 2012 by Sjors Gielen
All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

1\. Redistributions of source code must retain the above copyright notice, this
   list of conditions and the following disclaimer.
2\. Redistributions in binary form must reproduce the above copyright notice,
   this list of conditions and the following disclaimer in the documentation
   and/or other materials provided with the distribution.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND
ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT OWNER OR CONTRIBUTORS BE LIABLE FOR
ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
(INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND
ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
(INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
