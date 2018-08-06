# TiddlyWiki on SiT

This project is about using https://github.com/sit-fyi/sit as a backend for TiddlyWiki.

# Rationale

There at least two aspects where alternative backend storage is useful.

## Project scratch-pad which travels with code repository

We firmly believe that all software development artifacts must travel with the code.
Use TiddlyWiki for storing thought process about how and why certain feature were
developed makes total sense. However storing HTML file in the repository is not
practical since it makes PR reviews harder due to HTML noise.

TiddlyWiki has server implementation which is using `nodejs`. It is file based
storage which stores every tiddler in separate file. However it is quite
complicated and awkward to use. There are following problems with `nodejs`
implementation:

- manual addition of tiddler files into tiddlers directory wouldn't make
  them available in TiddlyWiki

## TiddlyWiki on smartphones

`nodejs` based installation of TiddliWiki on a smartphone is very involved process.
Alternative self contained backend might help here.

# Design

## Goals

1. To be able to save and retrieve tiddlers from sit
2. Take into account eventual support for
  - versioning of tiddlers
  - resolution of conflicts
3. Support queries via sit
  - who/what/when/how did the change
  - list all tiddlers with given tags
  - list of tiddlers with field matching given condition
4. Enable injection of tiddlers into wiki from CLI
5. Support for attachments (pdf, images, docs, video, audio)

## Gist of the approach

- create SyncAdaptor for sit (we would name it `sitadaptor`).
- use uuid in title
- use [unilink plugin](https://github.com/wikilabs/plugins/tree/master/wikilabs/uni-link)
  to display subtitle instead of title
- update new tiddler template to hide real title (if possible)

## Sit Records

- TWTagsAdded - Adds set of tags to tiddler. The tags are unordered.
- TWTagsRemoved - Removes set of tags from tiddler.
- TWFieldsAdded - Add set of fields to the tiddler
- TWFieldsRemoved - Remove set of fields from the tiddler
- TWTitleChanged - Change the title of the tiddler
- TWTypeChanged - Change the type of the tiddler
- TWBodyChanged - Change the body of the tiddler
- TWDeleted - Delete tiddler from the storage
- TWCommit - Commit a number of changes to tiddler
- TWAttachAdd - Attach an arbitrary file to tiddler
- TWAttachRemove - Remove attached file from tiddler

Changes in set of tags, fields or attachments are implemented via two
complimentary operations add/remove. We would use TWCommit record to make
the change atomic. The `TWCommit` implementation would rely on SiT ability
to have multiple references in `.prev/`.

### TWTagsAdded - Adds set of tags to the tiddler

Add set of tags to the tiddler. Tags to be added to tiddler are one per line
without any defined order among them.

- `./type/TWTagsAdded`
- `tags` - file containing tags to be added to the tiddler (one tag per line)
- `.authors` - the originator of the change
- `.timestamp` - the timestamp of the change
- `.version` - version of tiddlysit format used to create the record

### TWTagsRemoved - Remove set of tags from the tiddler

Remove set of tags from the tiddler. Tags to be removed from the tiddler are one
per line without any defined order among them.

- `./type/TWTagsRemoved`
- `tags` - file containing tags to be removed from the tiddler (one tag per line)
- `.authors` - the originator of the change
- `.timestamp` - the timestamp of the change
- `.version` - version of tiddlysit format used to create the record

### TWFieldsAdded - Add set of fields to the tiddler

Assign values to set of fields of the tiddler. The values are assigned via
assignment expressions one expression per line. Assignment expressions are
like following `<field_foo>=1234`.

- `./type/TWFieldsAdded`
- `fields` - file containing assignment expressions to be added to the tiddler
  (one expression per line)
- `.authors` - the originator of the change
- `.timestamp` - the timestamp of the change
- `.version` - version of tiddlysit format used to create the record


### TWFieldsRemoved - Remove set of fields from the tiddler

Remove fields from the tiddlers. The fields to remove are specified in the same
form as in TWFieldsAdded. I.e. one assignment expression per line. The values
in assignment expression are the current values known to TiddlyWiki.

- `./type/TWFieldsRemoved`
- `fields` - file containing assignment expressions for fields to be
  removed from the tiddler
- `.authors` - the originator of the change
- `.timestamp` - the timestamp of the change
- `.version` - version of tiddlysit format used to create the record

### TWTitleChanged - Change the title of the tiddler

- `./type/TWTitleChanged`
- `text` - file containing the diff of the change (as produced by GNU diff)
- `.authors` - the originator of the change
- `.timestamp` - the timestamp of the change
- `.version` - version of tiddlysit format used to create the record

### TWTypeChanged - Change the type of the tiddler

- `./type/TWTypeChanged`
- `type` - file containing the diff of the change (as produced by GNU diff).
  In case of a type it would contain
  ```
  -old_type
  +new_type
  ```
- `.authors` - the originator of the change
- `.timestamp` - the timestamp of the change
- `.version` - version of tiddlysit format used to create the record

### TWBodyChanged - Change the body of the tiddler

- `./type/TWTitleChanged`
- `text` - file containing the diff of the change (as produced by GNU diff)
- `.authors` - the originator of the change
- `.timestamp` - the timestamp of the change
- `.version` - version of tiddlysit format used to create the record

### TWDeleted - Delete tiddler from the storage

This record signifies the deletion of the tiddler. The deleted tiddlers wouldn't
be available to TiddlyWiki but it would be still present in SiT storage.

### TWCommit - Commit a number of changes to tiddler

This record relies on ability of SiT to have multiple references in `.prev`.
Essentially it makes the operation which involves multiple records atomic.

- `./type/TWCommit`
- `.prev/` - List of refences to records which constitute the operation
- `.authors` - the originator of the change
- `.timestamp` - the timestamp of the change
- `.version` - version of tiddlysit format used to create the record

### TWAttachAdd - Attach an arbitrary file to tiddler

TBD

### TWAttachRemove - Remove attached file from tiddler

TBD

## Migration

We would create a `sitadaptor` plugin home page which would have import from
HTML file button. We would provide instructions for user how to export their
wiki to HTML file and import it to SiT enabled edition. In `sitadaptor` we
would mangle the structure of the tiddler to facilitate the SiT operations
using following logic.

- If there is no `sit` field in the tiddler we would consider it to be not
  in SiT format. We would:
  - create new item in SiT
  - copy content of `title` field to `subtitle`
  - use `UUID` returned from SiT as `title`
  - store the current `tiddlysit` version in `sit`
  - store mangled tiddler in SiT
