# WASI Blob Store

A proposed [WebAssembly System Interface](https://github.com/WebAssembly/WASI) API.

### Current Phase

Phase 1

### Champions

- Jiaxiao Zhou
- Kevin Hoffman
- David Justice
- Dan Chiarlon
- Taylor Thomas

### Phase 4 Advancement Criteria

* [ ] At least two independent production implementations.
* [ ] At least two cloud provider implementations.
* [ ] Implementations available for at least Windows, Linux & MacOS.
* [ ] A test suite that passes on the platforms and implementations mentioned above.

## Table of Contents

- [Introduction](#introduction)
- [Goals](#goals)
- [Non-goals](#non-goals)
- [API walk-through](#api-walk-through)
  - [Process Blob Contents](#process-blob-contents)
  - [Write to a Blob Stream](#write-to-a-blob-stream)
  - [List Objects within a Container](#list-objects-within-a-container)
- [Detailed Design Discussion](#detailed-design-discussion)
  - [Handling Large Files](#handling-large-files)  
- [Stakeholder Interest & Feedback](#stakeholder-interest--feedback)

### Introduction

**Blob storage** is a type of data storage used for unstructured data such as images, videos, documents, backups, etc. Blob storage is also commonly referred to as _object storage_. The term **blob** is actually an acronym for **B**inary **L**arge **OB**ject but can be used to refer to all types of unstructured data.

Within the context of this proposal, blob storage refers to granting WebAssembly components access to a common abstraction of a blob store. Examples of blob storage services include [Azure Blob Storage](https://azure.microsoft.com/en-us/services/storage/blobs/), [AWS S3](https://aws.amazon.com/s3/), or [Google Cloud Storage](https://cloud.google.com/storage), but can be anything that can be represented as unstructured binary data that conforms to the interface, including file systems.

### Goals

The primary goal of this API is to provide a common abstraction for blob storage services, so that WebAssembly components can be written to work with any implementation, without needing to know the details of the underlying service.

Additionally, components using this API will be unable to tell the difference between a blob storage service and a file system, allowing them to be written to work with either and will not need to configure the store within the component code.

### Non-goals
The following is a list of goals explicitly out of scope for this API specification:

* Cover all edge cases and niche scenarios
* Configuration of service access
* Secrets management
* Definition or direct use of networking protocols
* Monitoring and Observability

### API walk-through

The following sections provide an overview of how this API might be used. Note that while the samples are in Rust, any language targetable by wasm components via code generation should work.

#### Process Blob Contents
This example shows obtaining a reference to the container and the desired object within that container, and then using `read_into` in a loop to access the blob contents.

```rust
// Count the number of lines in an object
// For simplicity, assume the object contains ascii text and lines end in '\n'
fn count_lines(store: &impl BlobStore, id: &ObjectId) -> Result<usize, Error> {
  let mut stream = store.get_container(&id.container_name)?.read_object(&id.object_name)?;
  let mut buf = [0u8; 4096];
  let mut num_lines = 0;
  while let Some(bytes) = stream.read_into(&mut buf)? {
    num_lines += buf[0..bytes as usize].iter().filter(|&c| *c == b'\n').count();
  }
  Ok(num_lines)
}
```

#### Write to a Blob Stream
The following code sample shows how to obtain a reference to a container and a writable reference to a stream that will be stored in a blob.

```rust
// Download a file from an http url and save it to the blob store.
// When completed, returns metadata for the new object
fn download(url: &str, store: &impl BlobStore, id: &ObjectId) -> Result<ObjectMetadata, Error> {
    let container = store.get_container(&id.container_name)?;
    // retrieve a url via wasi-http fetch() method
    // the http service hasn't been defined yet, but assume its fetch() method returns a readable stream.
    let mut download_stream = http::fetch(url)?;
    let mut buf = [0u8; 4096];
    let mut save_stream = container.write_object(&id.object_name)?;
    while let Some(bytes) = download_stream.read_into(&mut buf)? {
        save_stream.write(&buf[0..bytes as usize])?;
    }
    // ensure stream is flushed and object is created, before we query the metadata
    save_stream.close()?;
    let obj = container.object_info(&id.object_name)?;
    Ok(obj)
}
```

#### List Objects within a Container
The following code shows how to enumerate the objects within a container.

```rust
// suppose the "logs" container has objects with names that start with a timestamp, like "2022-01-01-12-00-00.log"
// for every day that activity occurred. To count the number of logs from january 2022, call:
//    `count_objects_with_prefix(store, "logs", "2022-01")`
fn count_objects_with_prefix(store: &impl BlobStore, container_name: &str, prefix: &str) -> Result<usize,Error> {
  let container = store.get_container(container_name)?;
  let names = container.list_objects()?;
  let count = names.filter(|n| n.starts_with(prefix)).count();
  Ok(count)
}
```

### Detailed Design Discussion

See the [wit files](./wit)

#### Handling Large Files

Handling large files may require changes to the API that are not accounted for in this current proposal. If a component attempts to allocate more memory than the host is willing to give it, then the component could be terminated by the host runtime and the processing will fail.

Additionally, if a component spends too long processing a file, either processing one large blob or by processing many small blobs in a tight loop, then the component could again be shut off because it consumed too many resources or too much time.

How to handle this and whether the `callback` approach belongs in the blob store API or in a lower level [wasm-io API](https://github.com/WebAssembly/wasi-io/issues/31) is still under discussion.

### Stakeholder Interest & Feedback

TODO before entering Phase 3.
