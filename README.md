# Conflux S3 Utils

A Python library providing type-safe interfaces and convenient utilities for interacting with objects in S3.

## Installation

```
pip install conflux-s3-utils
```

## Usage

The primitives in this library are [`S3Uri`](./conflux_s3_utils/s3uri.py) and [`S3Client`](./conflux_s3_utils/s3client.py).

* `S3Uri` provides a type-safe way to specify S3 URIs, in the spirit of [parse, don't validate](https://lexi-lambda.github.io/blog/2019/11/05/parse-don-t-validate/).
* `S3Client` is a client that provides convenient methods for interacting with S3 objects specified via their `S3Uri`.