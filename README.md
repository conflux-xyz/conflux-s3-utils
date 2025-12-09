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

### `S3Uri`

```python
s3uri_str = "s3://bucket/path/to/object.txt"
# Parse from a S3 URI string
s3uri = S3Uri.from_str(s3uri_str)
# Check the bucket and path (key)
assert s3uri.bucket == "bucket"
assert s3uri.path == "path/to/object.txt"
# Get just the filename
assert s3uri.filename == "object.txt"
# Convert back into a S3 URI string
assert str(s3uri) == "s3://bucket/path/to/object.txt"

# Replace the path
s3uri_dir = s3uri.with_path("object/directory")
# Check the bucket and path (key)
assert s3uri_dir.bucket == "bucket"
assert s3uri_dir.path == "object/directory"

# Use slash operator for child paths
child_s3uri = s3uri_dir / "subfolder" / "foo.c"
assert child_s3uri.bucket == "bucket"
assert child_s3uri.path == "object/directory/subfolder/foo.c"
assert str(child_s3uri) == "s3://bucket/object/directory/subfolder/foo.c"
```