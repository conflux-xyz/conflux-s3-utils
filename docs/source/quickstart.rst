Quick Start
===========

This guide will help you get started with Conflux S3 Utils quickly.

Basic Usage
-----------

Initializing the Client
~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   from conflux_s3_utils import S3Client

   # Create a client with default boto3 configuration
   s3 = S3Client()

   # Or provide your own boto3 client
   import boto3
   custom_client = boto3.client('s3', region_name='us-west-2')
   s3 = S3Client(client=custom_client)

Working with S3 URIs
~~~~~~~~~~~~~~~~~~~~

S3Uri provides a type-safe way to work with S3 URIs:

.. code-block:: python

   from conflux_s3_utils import S3Uri

   # Parse from a S3 URI string
   s3uri = S3Uri.from_str("s3://bucket/path/to/object.txt")

   # Access bucket and path components
   print(s3uri.bucket)  # "bucket"
   print(s3uri.path)    # "path/to/object.txt"
   print(s3uri.filename)  # "object.txt"

   # Convert back to a S3 URI string
   print(str(s3uri))  # "s3://bucket/path/to/object.txt"

Path Manipulation
~~~~~~~~~~~~~~~~~

S3Uri supports convenient path manipulation:

.. code-block:: python

   # Replace the entire path
   s3uri_dir = s3uri.with_path("new/path/directory")

   # Join paths using the slash operator
   child_s3uri = s3uri_dir / "subfolder" / "file.txt"
   # Result: s3://bucket/new/path/directory/subfolder/file.txt

   # Join paths using join_path method
   child_s3uri = s3uri_dir.join_path("subfolder", "file.txt")

Common Operations
-----------------

Opening Files
~~~~~~~~~~~~~

.. code-block:: python

   # Open an S3 object directly using fsspec
   s3uri = S3Uri.from_str("s3://bucket/data.json")

   # Read mode
   with s3.open(s3uri, mode="rb") as f:
       data = f.read()

   # Write mode
   with s3.open(s3uri, mode="wb") as f:
       f.write(b"Hello, S3!")

Working with Local Files
~~~~~~~~~~~~~~~~~~~~~~~~~

The ``open_local`` method is useful when you need a local file path:

.. code-block:: python

   # Read mode: Download from S3, work with local file
   s3uri = S3Uri.from_str("s3://bucket/data.csv")
   with s3.open_local(s3uri, mode="rb") as f:
       # File is downloaded to a temporary location
       data = f.read()
   # Temporary file is automatically cleaned up

   # Write mode: Work with local file, upload to S3
   with s3.open_local(s3uri, mode="wb") as f:
       f.write(b"New data")
   # File is automatically uploaded to S3 and cleaned up

Checking Object Existence
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   s3uri = S3Uri.from_str("s3://bucket/path/to/object.txt")
   if s3.object_exists(s3uri):
       print("Object exists!")

Listing Objects
~~~~~~~~~~~~~~~

.. code-block:: python

   # List objects directly under a path (non-recursive)
   s3uri = S3Uri.from_str("s3://bucket/path/to/directory")
   for obj_uri in s3.list_objects(s3uri):
       print(obj_uri)

   # List all objects recursively
   for obj_uri in s3.list_objects(s3uri, recursive=True):
       print(obj_uri)

Uploading and Downloading
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   from pathlib import Path

   # Upload a local file
   local_file = Path("./local/data.txt")
   s3uri = S3Uri.from_str("s3://bucket/remote/data.txt")
   s3.upload_file(local_file, s3uri)

   # Download a file
   s3.download_file(s3uri, local_file)

   # Customize multipart upload/download chunk size (default 8MB)
   s3.upload_file(local_file, s3uri, multipart_chunksize=16*1024*1024)  # 16MB chunks

Uploading Directories
~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   from pathlib import Path

   # Upload an entire directory
   local_dir = Path("./local/directory")
   s3uri = S3Uri.from_str("s3://bucket/remote/directory")
   s3.upload_directory(local_dir, s3uri)

   # Upload with custom concurrency (default is 10 concurrent uploads)
   s3.upload_directory(local_dir, s3uri, concurrency=5)

   # Exclude specific files
   exclude = {Path("secret.txt"), Path("cache/temp.dat")}
   s3.upload_directory(local_dir, s3uri, exclude=exclude)

Copying and Deleting
~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   # Copy an object within S3
   src = S3Uri.from_str("s3://bucket/source/file.txt")
   dest = S3Uri.from_str("s3://bucket/destination/file.txt")
   s3.copy_object(src, dest)

   # Delete an object
   s3.delete_object(s3uri)

Complete Example
----------------

Here's a complete example that demonstrates common operations:

.. code-block:: python

   from pathlib import Path
   from conflux_s3_utils import S3Client, S3Uri

   # Initialize client
   s3 = S3Client()

   # Parse S3 URIs
   base_uri = S3Uri.from_str("s3://my-bucket/data")
   input_uri = base_uri / "input" / "data.csv"
   output_uri = base_uri / "output" / "results.txt"

   # Check if input exists
   if not s3.object_exists(input_uri):
       raise ValueError(f"Input file not found: {input_uri}")

   # Download and process
   local_input = Path("./temp/input.csv")
   s3.download_file(input_uri, local_input)

   # Process the file
   with open(local_input, "r") as f:
       processed_data = f.read().upper()

   # Upload results
   local_output = Path("./temp/output.txt")
   with open(local_output, "w") as f:
       f.write(processed_data)
   s3.upload_file(local_output, output_uri)

   print(f"Results uploaded to: {output_uri}")

Next Steps
----------

* Check out the :doc:`api` for detailed API documentation
* See more :doc:`examples` for advanced usage patterns
