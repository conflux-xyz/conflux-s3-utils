Examples
========

This page provides more advanced examples of using Conflux S3 Utils.

Data Processing Pipeline
-------------------------

This example shows how to build a data processing pipeline that reads from S3, processes data, and writes back to S3:

.. code-block:: python

   import json
   from pathlib import Path
   from conflux_s3_utils import S3Client, S3Uri

   def process_json_files(bucket: str, input_prefix: str, output_prefix: str):
       """Process all JSON files in an S3 prefix and write results."""
       s3 = S3Client()

       # List all JSON files
       input_base = S3Uri.from_str(f"s3://{bucket}/{input_prefix}")
       output_base = S3Uri.from_str(f"s3://{bucket}/{output_prefix}")

       for obj_uri in s3.list_objects(input_base, recursive=True):
           if not obj_uri.filename.endswith('.json'):
               continue

           print(f"Processing {obj_uri}")

           # Read and process
           with s3.open(obj_uri, mode="rb") as f:
               data = json.load(f)

           # Transform data (example: add a field)
           data['processed'] = True
           data['processed_by'] = 'pipeline_v1'

           # Write to output
           relative_path = obj_uri.path.removeprefix(input_prefix).lstrip('/')
           output_uri = output_base.join_path(relative_path)

           with s3.open(output_uri, mode="wb") as f:
               f.write(json.dumps(data, indent=2).encode())

           print(f"Wrote to {output_uri}")

   # Usage
   process_json_files("my-bucket", "raw/data", "processed/data")

Batch Upload with Progress Tracking
------------------------------------

This example shows how to upload multiple files with progress tracking:

.. code-block:: python

   from pathlib import Path
   from conflux_s3_utils import S3Client, S3Uri

   def upload_with_progress(local_dir: Path, s3_prefix: str):
       """Upload directory with progress tracking."""
       s3 = S3Client()
       base_uri = S3Uri.from_str(s3_prefix)

       # Collect all files
       files = list(local_dir.rglob("*"))
       files = [f for f in files if f.is_file()]

       print(f"Uploading {len(files)} files...")

       for i, filepath in enumerate(files, 1):
           relative_path = filepath.relative_to(local_dir)
           s3uri = base_uri.join_path(str(relative_path))

           s3.upload_file(filepath, s3uri)
           print(f"[{i}/{len(files)}] Uploaded {relative_path}")

       print("Upload complete!")

   # Usage
   upload_with_progress(Path("./data"), "s3://my-bucket/uploads")

Concurrent File Processing
---------------------------

Process multiple S3 files concurrently:

.. code-block:: python

   import concurrent.futures
   from conflux_s3_utils import S3Client, S3Uri

   def process_file(s3: S3Client, s3uri: S3Uri) -> dict:
       """Process a single file and return results."""
       with s3.open(s3uri, mode="rb") as f:
           content = f.read()

       # Your processing logic here
       result = {
           'uri': str(s3uri),
           'size': len(content),
           'processed': True
       }
       return result

   def process_files_concurrent(bucket: str, prefix: str, max_workers: int = 10):
       """Process multiple S3 files concurrently."""
       s3 = S3Client()
       base_uri = S3Uri.from_str(f"s3://{bucket}/{prefix}")

       # Get all files
       files = list(s3.list_objects(base_uri, recursive=True))

       print(f"Processing {len(files)} files with {max_workers} workers...")

       # Process concurrently
       with concurrent.futures.ThreadPoolExecutor(max_workers=max_workers) as executor:
           futures = {executor.submit(process_file, s3, uri): uri for uri in files}

           results = []
           for future in concurrent.futures.as_completed(futures):
               uri = futures[future]
               try:
                   result = future.result()
                   results.append(result)
                   print(f"Processed {uri}")
               except Exception as e:
                   print(f"Error processing {uri}: {e}")

       return results

   # Usage
   results = process_files_concurrent("my-bucket", "data/to/process")

Working with Large Files
-------------------------

Handle large files efficiently using local temporary files:

.. code-block:: python

   import gzip
   from conflux_s3_utils import S3Client, S3Uri

   def compress_large_file(source_uri: str, dest_uri: str):
       """Download a large file, compress it, and upload."""
       s3 = S3Client()
       source = S3Uri.from_str(source_uri)
       dest = S3Uri.from_str(dest_uri)

       # Use 16MB chunks for large files
       chunk_size = 16 * 1024 * 1024

       # Read from S3, compress, write to S3
       with s3.open_local(source, mode="rb", multipart_chunksize=chunk_size) as input_file:
           with s3.open_local(dest, mode="wb", multipart_chunksize=chunk_size) as output_file:
               with gzip.open(output_file, 'wb') as gz:
                   # Process in chunks to avoid memory issues
                   while True:
                       chunk = input_file.read(chunk_size)
                       if not chunk:
                           break
                       gz.write(chunk)

       print(f"Compressed {source} to {dest}")

   # Usage
   compress_large_file(
       "s3://my-bucket/large-file.txt",
       "s3://my-bucket/large-file.txt.gz"
   )

Conditional Operations
----------------------

Perform operations based on object existence and properties:

.. code-block:: python

   from conflux_s3_utils import S3Client, S3Uri

   def safe_copy_with_backup(source_uri: str, dest_uri: str):
       """Copy a file, backing up the destination if it exists."""
       s3 = S3Client()
       source = S3Uri.from_str(source_uri)
       dest = S3Uri.from_str(dest_uri)

       # Check if destination exists
       if s3.object_exists(dest):
           # Create backup
           backup = dest.with_path(f"{dest.path}.backup")
           print(f"Backing up {dest} to {backup}")
           s3.copy_object(dest, backup)

       # Perform the copy
       print(f"Copying {source} to {dest}")
       s3.copy_object(source, dest)

   # Usage
   safe_copy_with_backup(
       "s3://my-bucket/source/file.txt",
       "s3://my-bucket/dest/file.txt"
   )

Directory Synchronization
--------------------------

Sync a local directory to S3, excluding certain files:

.. code-block:: python

   from pathlib import Path
   from conflux_s3_utils import S3Client, S3Uri

   def sync_directory(local_dir: Path, s3_prefix: str, exclude_patterns: list[str]):
       """Sync a local directory to S3, excluding files matching patterns."""
       s3 = S3Client()
       base_uri = S3Uri.from_str(s3_prefix)

       # Build exclude set
       exclude = set()
       for pattern in exclude_patterns:
           exclude.update(local_dir.rglob(pattern))

       # Convert to relative paths
       exclude_relative = {f.relative_to(local_dir) for f in exclude}

       print(f"Syncing {local_dir} to {s3_prefix}")
       print(f"Excluding {len(exclude_relative)} files")

       # Upload with exclusions
       s3.upload_directory(
           local_dir,
           base_uri,
           exclude=exclude_relative,
           concurrency=10
       )

       print("Sync complete!")

   # Usage
   sync_directory(
       Path("./my-project"),
       "s3://my-bucket/backups/my-project",
       exclude_patterns=["*.pyc", "__pycache__", ".git/*", "*.log"]
   )

Custom Boto3 Configuration
---------------------------

Use custom boto3 configuration for specific needs:

.. code-block:: python

   import boto3
   from botocore.config import Config
   from conflux_s3_utils import S3Client

   # Create custom boto3 client with specific configuration
   config = Config(
       region_name='us-west-2',
       signature_version='s3v4',
       retries={
           'max_attempts': 10,
           'mode': 'adaptive'
       },
       max_pool_connections=50
   )

   boto3_client = boto3.client('s3', config=config)
   s3 = S3Client(client=boto3_client)

   # Now use s3 with your custom configuration
   # This is useful for:
   # - Specific region requirements
   # - Custom retry logic
   # - Connection pool tuning
   # - Signature version requirements
