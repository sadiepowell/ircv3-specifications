---
title: Extended Ban ISUPPORT
layout: spec
copyrights:
  -
    name: "Sadie Powell"
    email: "sadie@sadiepowell.dev"
    period: "2026"
---

## Introduction

Extended bans are a de-facto standard that allows list mode entries to be used to define custom matching behaviour. Recently IRC server vendors have augmented extended bans in ways that are not currently communicated to clients.

This specification defines a formal method of communicating this behaviour to clients via the use of [ISUPPORT][] tokens similar to the existing [EXTBAN][] token.

[EXTBAN]: https://modern.ircdocs.horse/#extban-parameter
[ISUPPORT]: https://modern.ircdocs.horse/#feature-advertisement

## ISUPPORT tokens

### The `EXTBANFORMAT` token

Servers publishing the `EXTBANFORMAT` ISUPPORT token declare the normalised format that they use for storing extended bans.

If the `EXTBANFORMAT` token is declared then it must have one of the following values:

Value  | Description
------ | -----------
any    | Extended bans are stored in the format that they are sent to the server.
letter | Extended bans are normalised to the letter format.
name   | Extended bans are normalised to the name format.

#### Examples

Setting an extended ban using the any format:

    S: :irc.ircv3.net 005 yoko ACCOUNTEXTBAN=account,R EXTBAN=~,R EXTBANFORMAT=any :are supported by this server
    ...
    C: MODE #japan +bb ~R:airi ~account:chitose


Setting an extended ban using the letter format:

    S: :irc.ircv3.net 005 yoko ACCOUNTEXTBAN=R EXTBAN=~,R EXTBANFORMAT=letter :are supported by this server
    ...
    C: MODE #japan +b ~R:airi



Setting an extended ban using the name format:

    S: :irc.ircv3.net 005 yoko ACCOUNTEXTBAN=account EXTBAN=~, EXTBANFORMAT=name :are supported by this server
    ...
    C: MODE #japan +b ~account:chitose


### The `EXTBANINVERT` token

Servers publishing the `EXTBANINVERT` ISUPPORT token declare whether they support inverted extended bans. Inverted extended bans invert the usual matching behaviour of a ban, e.g. on InspIRCd `!m:*!*@example.com` would mute all users who do not have the hostname `example.com`..

If the `EXTBANINVERT` token is declared with no value then a server is explicitly declaring non-support for inverted extended bans. Otherwise, the value of the token is the character sequence used after the extended ban prefix and before the extended ban value to invert the value.

#### Examples

A server with the extban prefix `$ ` declaring that it supports inverted extbans with the prefix `~`:

    S: :irc.ircv3.net 005 sorawo ACCOUNTEXTBAN=R EXTBAN=$,R EXTBANINVERT=~ :are supported by this server
    ...
    C: MODE #otherside +b $~R:toriko

A server with no extban prefix declaring that it supports inverted extbans:

    S: :irc.ircv3.net 005 toriko ACCOUNTEXTBAN=R EXTBAN=,R EXTBANINVERT=! :are supported by this server
    ...
    C: MODE #otherside +b !R:sorawo

### The `EXTBAN/` prefix

The `EXTBAN` ISUPPORT prefix is reserved for named extended bans. For each named extended ban the server SHOULD declare an ISUPPORT token with the `EXTBAN/` prefix followed by the name. If the server has also declared a character equivalent in the `EXTBAN` token the value of the ISUPPORT token MUST be set to the equivalent character. Servers SHOULD declare the latter half of the token in the canonical case of the named extended ban.

#### Example

A server which supports [account-based bans](https://modern.ircdocs.horse/#extban-parameter) using the name `account` and the character `R`:

    S: irc.ircv3.net 005 maomao ACCOUNTEXTBAN=account,R EXTBAN=,R EXTBAN/account=R :are supported by this server

## Implementation Considerations

Clients SHOULD be aware that servers may internally normalise the structure of an extended ban and SHOULD NOT expect that they will receive a `MODE` message with the exact same mode parameter as the one they sent to the server. To avoid issues relating to normalisation it is recommended that clients use the [`labeled-response`](./labeled-response.html) specification to map a mode change to the associated server response.

Servers SHOULD be permissive regarding set extended bans. If a user sends an extended ban in a format different to that of `ISUPPORTFORMAT` then the server SHOULD normalise the extended ban to the correct format.

Servers SHOULD normalise set named extended bans to their canonical case using the IRC casemapping system for consistency. Servers SHOULD also not allow two functionally identical extended bans with different syntax to be set.
