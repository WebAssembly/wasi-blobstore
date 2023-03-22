```

// # wasi-cloud blobstore service definition
//
// A blobstore is service that can mange containers and objects.
//
// ## Object
// An object is a named sequence of bytes. Objects can be read or written with a stream interface.
// Once an object is written its contents cannot be modified.
// Objects can be any size from 1 byte up to the limits imposed by the underlying store.
// Object names are unique within their container.
//
// ## Container
// A container is a named collection of objects. Container names can be any utf-8 string.
// Within any instance of the blobstore interface, all containers share the same "flat" namespace.


// wasi-cloud Blobstore service definition
interface "wasi:blob/blobstore" {

  use wasi:blob/types::{ Error, container-metadata, container-name, object-id }

  // creates a new empty container
  create-container: func(name: container-name) -> result<container,Error>

  // retrieves a container by name
  get-container: func(name: container-name) -> result<container, Error>

  // deletes a container and all objects within it
  delete-container: func(name: container-name) -> result<_, Error>

  // returns true if the container exists
  container-exists: func(name: container-name) -> result<bool, Error>

  // copies (duplicates) an object, to the same or a different container.
  // returns an error if the target container does not exist.
  // overwrites destination object if it already existed.
  copy-object: func(src: object-id, dest: object-id) -> result<_, Error>

  // moves or renames an object, to the same or a different container
  // returns an error if the destination container does not exist.
  // overwrites destination object if it already existed.
  move-object: func(src:object-id, dest: object-id) -> result<_, Error>
}

// a Container is a collection of objects
resource "wasi:blob/container" {

  // returns container name
  name: func() -> result<string, Error>

  // returns container metadata
  info: func() -> result<container-metadata, Error>

  // begins reading an object
  read-object: func(name: object-name) -> result<read-stream, Error>

  // creates or replaces an object.
  write-object: func(name: object-name) -> result<write-stream, Error>

  // retrieves an object or portion of an object, as a resource.
  // Start and end offsets are inclusive.
  // Once a data-blob resource has been created, the underlying bytes are held by the blobstore service for the lifetime
  // of the data-blob resource, even if the object they came from is later deleted.
  get-data: func(name: object-name, start: u64, end: u64) -> result<data-blob, Error>

  // creates or replaces an object with the data blob.
  write-data: func(name: object-name, data: data-blob) -> result<_, Error>

  // returns list of objects in the container. Order is undefined.
  list-objects: func(name: object-name) -> result<stream<object-name>, Error>

  // deletes object.
  // does not return error if object did not exist.
  delete-object: func(name: object-name) -> result<_, Error>

  // deletes multiple objects in the container
  delete-objects: func(names: list<object-name>) -> result<_, Error>

  // returns true if the object exists in this container
  has-object: func(name: object-name) -> result<bool, Error>

  // returns metadata for the object
  object-info: func(name: object-name) -> result<object-metadata, Error>

  // removes all objects within the container, leaving the container empty.
  clear: func() -> result<_, Error>
}

// A write stream for saving an object to a blobstore.
resource "wasi:blob/write-stream" {

  // writes (appends) bytes to the object.
  write: func(data: list<u8>) -> result<_, Error>

  // closes the write stream
  close: func() -> result<_,Error>
}

// A read stream for retrieving an object (or object region) from blob store
resource "wasi:blob/read-stream" {

  // reads bytes from the object into an existing array,
  // until the buffer is full or the end of the stream.
  // Returns number of bytes written, or none if the stream has ended.
  read-into: func(ref mut list<u8>) -> result<option<u64>, Error>

  // returns the number of bytes remaining that could be read until the end of the stream.
  available: func() -> result<u64, Error>

  // closes the read stream. May be used by reader to signal that it is not interested in reading more bytes.
  close: func() -> result<_, Error>
}

// A data-blob resource references a byte array. It is intended to be lightweight
// and can be passed to other components, without the overhead of copying the underlying bytes.
// A data-blob can be created with object::get-data(), or with the create() function below.
resource "wasi:blob/data-blob" {

  // creates a new data blob
  create: func() -> data-blob-writer

  // begins reading this data-blob
  read: func() -> result<read-stream, Error>

  // returns the total size of this data-blob
  size: func() -> result<u64, Error>
}

// A data-blob-writer is a writable stream that creates a transient data-blob.
// The data-blob can later be saved to an object with container::write-data()
resource "wasi:blob/data-blob-writer" {

  // append bytes to a data-blob
  write: func(data: list<u8>) -> result<_, Error>

  // finish writing blob
  finalize: func() -> result<data-blob, Error>
}

// Types used by blobstore
interface "wasi:blob/types" {

  // name of a container, a collection of objects.
  // The container name may be any valid UTF-8 string.
  type container-name = string

  // name of an object within a container
  // The object name may be any valid UTF-8 string.
  type object-name = string

  // information about a container
  record container-metadata {
    // the container's name
    name: container-name,
    // date and time container was created
    created-at: timestamp,
  }

  // information about an object
  record object-metadata {
    // the object's name
    name: object-name,
    // the object's parent container
    container: container-name,
    // date and time the object was created
    created-at: timestamp,
    // size of the object, in bytes
    size: u64,
  }

  // identifier for an object that includes its container name
  record object-id {
    container: container-name,
    object: object-name
  }
}
```
