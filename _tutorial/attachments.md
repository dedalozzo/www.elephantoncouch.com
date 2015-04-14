---
layout: tutorial
title: Attachments
permalink: /tutorial/attachments/
---

Attachments store data associated with a document, but are not part of the document's JSON object. Their primary purpose is to make it efficient to store large binary data in a document. Binary data stored in JSON has to be base64-encoded into a string, which inflates its size by 33%. Also, binary data blobs are often large (think of camera images or audio files), and big JSON documents are slow to parse.

Attachments are uninterpreted data (blobs) stored separately from the JSON body. A document can have any number of attachments, each with a different name. Each attachment is also tagged with a MIME type, which isn't used by Couchbase Lite but can help your application interpret its contents. Attachments can be arbitrarily large, and are only read on demand, not when you load a Document object.

Attachments also make replication more efficient. When a document that contains pre-existing attachments is synced (to the server or to a client), only attachments that have changed since the last sync are transferred over the network. In particular, changes to document JSON values will not cause Couchbase Lite to re-send attachment data when the attachment has not changed.

In the native API, attachments are represented by the Attachment class. Attachments are available from a Revision object. From a Document, you get to the attachments via its currentRevision.

# Reading attachments

The Revision class has a number of methods for accessing attachments:

attachmentNames returns the names of all the attachments.
attachmentNamed returns an Attachment object given its name.
attachments returns all the attachments as Attachment objects.
Once you have an Attachment object, you can access its name, MIME type and content length. The accessors for the content vary by platform: on iOS it's available as an NSData object or as an NSURL pointing to a read-only file; in Java you read the data from an InputStream.

## Attachment storage

In general, you don't need to think about where and how Couchbase Lite is storing data. But since attachments can occupy a lot of space, it can be helpful to know where that space is and how it's managed.

Attachments aren't stored in the database file itself. Instead they are individual files, contained in a directory right next to the database file. Each attachment file has a cryptic name that is actually a SHA-1 digest of its contents.

As a consequence of the naming scheme, attachments are de-duplicated: if multiple attachments in the same database have exactly the same contents, the data is only stored once in the filesystem.

Updating a document's attachment does not immediately remove the old version of the attachment. And deleting a document does not immediately delete its attachments. An attachment file has to remain on disk as long as there are any document revisions that reference it, And a revision persists until the next database compaction after it's been replaced or deleted. (Orphaned attachment files are deleted from disk as part of the compaction process.) So if you're concerned about the space taken up by attachments, you should compact the database frequently, or at least after making changes to large attachments.