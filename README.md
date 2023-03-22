# [Example WASI proposal]

This template can be used to start a new proposal, which can then be proposed in the WASI Subgroup meetings.

The sections below are recommended. However, every proposal is different, and the community can help you flesh out the proposal, so don't block on having something filled in for each one of them.

Thank you to the W3C Privacy CG for the [inspiration](https://github.com/privacycg/template)!

# [Title]

A proposed [WebAssembly System Interface](https://github.com/WebAssembly/WASI) API.

### Current Phase

[Fill in the current phase, e.g. Phase 1]

### Champions

- [Champion 1]
- [Champion 2]
- [etc.]

### Phase 4 Advancement Criteria

TODO before entering Phase 2.

## Table of Contents [if the explainer is longer than one printed page]

- [Introduction](#introduction)
- [Goals [or Motivating Use Cases, or Scenarios]](#goals-or-motivating-use-cases-or-scenarios)
- [Non-goals](#non-goals)
- [API walk-through](#api-walk-through)
  - [Use case 1](#use-case-1)
  - [Use case 2](#use-case-2)
- [Detailed design discussion](#detailed-design-discussion)
  - [[Tricky design choice 1]](#tricky-design-choice-1)
  - [[Tricky design choice 2]](#tricky-design-choice-2)
- [Considered alternatives](#considered-alternatives)
  - [[Alternative 1]](#alternative-1)
  - [[Alternative 2]](#alternative-2)
- [Stakeholder Interest & Feedback](#stakeholder-interest--feedback)
- [References & acknowledgements](#references--acknowledgements)

### Introduction

[The "executive summary" or "abstract". Explain in a few sentences what the goals of the project are, and a brief overview of how the solution works. This should be no more than 1-2 paragraphs.]

### Goals [or Motivating Use Cases, or Scenarios]

[What is the end-user need which this project aims to address?]

### Non-goals

[If there are "adjacent" goals which may appear to be in scope but aren't, enumerate them here. This section may be fleshed out as your design progresses and you encounter necessary technical and other trade-offs.]

### API walk-through

[Walk through of how someone would use this API.]

#### [Use case 1]

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

#### [Use case 2]

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

#### [Use case 3]

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

### Detailed design discussion

see [blobstore.md](./blobstore.wit.md)


#### [Tricky design choice #1]

[Talk through the tradeoffs in coming to the specific design point you want to make.]

```
// Illustrated with example code.
```

[This may be an open question, in which case you should link to any active discussion threads.]

#### [Tricky design choice 2]

[etc.]

### Considered alternatives

[This section is not required if you already covered considered alternatives in the design discussion above.]

#### [Alternative 1]

[Describe an alternative which was considered, and why you decided against it.]

#### [Alternative 2]

[etc.]

### Stakeholder Interest & Feedback

TODO before entering Phase 3.

[This should include a list of implementers who have expressed interest in implementing the proposal]

### References & acknowledgements

Many thanks for valuable feedback and advice from:

- [Person 1]
- [Person 2]
- [etc.]
