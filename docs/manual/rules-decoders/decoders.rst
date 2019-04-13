.. _manual_rules-decoders_decoders:

Decoders
========

OSSEC decoders attempt to identify the type of log message being inspected,
parse the message, and extract information to be used by the rules. 

The decoders that ship with OSSEC are in `/var/ossec/etc/decoder.xml`, and 
custom decoders can be added to `/var/ossec/etc/local_decoder.xml`.

Pre-decoding
------------

Each log message received by `ossec-analysisd` is pre-decoded. Certain information
is removed from the log message, some of it saved for later use.

The message's timestamp is removed. OSSEC does not currently use this information or
make it available.

For syslog messages, the name of the program that created the log message is availble
in the `program_name` field. If a `pid` is included in the log message, it is discarded.

The rest of the log message is then available to the decoders and rules for prossessing.

Parent and Children
-------------------

A decoder may be stand alone, a parent, or a child. A decoder may not be both a child and 
parent at the same time. 

Decoder Format
--------------

OSSEC decoders can be very simple or very complex, but the basic format
remains the same.

Simple example:

.. code-block:: xml

   <decoder name="sshd">
      <program_name>^sshd</program_name>
   </decoder>

Each decoder must have a name. A short descriptive name is probably best.

If the decoder is a child of another decoder (the child relies on the parent matching
first), a `<child>` option can be added. The value of this field is the name of the parent
decoder.

Each decoder must also have an element to differentiate the logs it understands
from other logs. The `sshd` decoder above uses `program_name` to do this. This field uses
the simple regex syntax.

`program_name_pcre2` can be used to utilize pcre2 syntax in the `program_name` field.

If the `program_name` is available it is the preferred method for matching a log message.

The option `<prematch>` can also be used to match a log message to a decoder.
The prematch value is a string that is matched to the log message (after the meta data,
like timestamp, is removed).

Much like `program_name`, `prematch` has a pcre2 version: `prematch_pcre2`.

To gather informationi into fields for rules decoders use `<regex>` or `<pcre2>`.
Any matches with regex or pcre2 can be saved to fields defined by the `<order> option,
separated by commas.

By default there are only a few fields available to `order`:

.. code-block:: console

   - location - where the log came from (only on FTS)
   - srcuser  - extracts the source username
   - dstuser  - extracts the destination (target) username
   - user     - an alias to dstuser (only one of the two can be used)
   - srcip    - source ip
   - dstip    - dst ip
   - srcport  - source port
   - dstport  - destination port
   - protocol - protocol
   - id       - event id
   - url      - url of the event
   - action   - event action (deny, drop, accept, etc)
   - status   - event status (success, failure, etc)
   - extra_data     - Any extra data

.. versionadded:: 3.3

As of version 3.3, OSSEC can create arbitrary fields, so those listed above are not 
the only ones available. Nested fields can be used to create nested fields in the json
output.


`<fts>` can be used to alert on first time seen information. The values for this option
should be the fields matched from `regex` or `pcre2` that should be tracked. It is common
to alert on the first instance of users escalating their privileges or an IP successfully
logging into a system for the first time.


