# DKIMrelay: Add a DKIM signature while relaying email

> *A straightforward SMTP filter adding DKIM signatures.*

This is a SMTP service that forwards emails to another SMTP service.
While in transit, it adds DKIM signatures.


## Adding DKIM

This filter can be configured to add a `DKIM-Signature:` header to
selected sending domains.  This will mostly be set to domain names
hosted locally.

The best place to add DKIM to an email is in the outward path.
This is a different approach than followed with OpenDKIM, which is
a milter, so incoming, mail filter.

When used with Postfix, the advised use is to insert it as a
`relay_host` or forward traffic through it via the `transport_maps`.
These are outgoing paths.  The assumption made in this tool is
that an upstream mail provider actually takes care of delivery.

Especially with Postfix, this approach means that changes made in
the various `canonical_maps` and in the `generic_maps` are made
before signing takes place.  Since these rewrite headers that are
commonly included in the `DKIM-Signature:` header, this is important
to retain a valid signature on a maximally diverse configuration.

When neither of these things are used, you may consider `content_filter`
setups with DKIMrelay.

Note that an MTA splits the delivery of a message, and so there may
be more than one outgoing path.  Since the DKIMrelay sits on the
outwards SMTP path, it *will not* sign locally delivered email.
So, if you have the habit of sending yourself a `Bcc:` to a local
address you will not see the added `DKIM-Signature:` header in it.


## Configuration

The configuration variables are set in an INI-style file, with
an example in `dkimrelay.conf` and the normal storage location in
`/etc/arpa2mail/dkimrelay.conf`.

There is a database management utility `dkimrelay-recipient-subject-tag`
that lists, clears or sets a database entry with 0, 1 and 2 arguments,
respectively.


## Relation with SRSrelay

We also created an
[SRSrelay](https://github.com/arpa2/srsrelay)
with a similar structure.  The two can only be combined when SRSrelay
runs *after* DKIMrelay.


## Adding Subject Tags

One problem with DKIM is that signatures break in various email lists.
This is usually caused by added text, especially a `[ListName]` tag
prepended to the `Subject:` header.

This is easily remedied, because those lists will not add the tag when
it is already inserted.  So, following that same logic, we can add the
tag ourselves just before sending over email.

To do this, a database mapping from (list) recipient addresses to
subject tags can be created with `dkimrelay-recipient-subject-tag`.
The same mapping applies to all users, and all domains, that pass
through the DKIMrelay.

When multiple recipients require tagging, all their tags will be
added.  But, as lists do, when a tag is found in the `Subject:`
header, it is not inserted again.  So the tagging inserted is
limited by reason.

If you don't setup this facility, it should not make any changes to
your email.
