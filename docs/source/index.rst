Conflux S3 Utils
================

A Python library providing type-safe interfaces and convenient utilities for interacting with objects in S3.

.. toctree::
   :maxdepth: 2
   :caption: Contents:

   installation
   quickstart
   api
   examples

Overview
--------

Conflux S3 Utils provides two main primitives for working with Amazon S3:

* **S3Uri**: A type-safe way to specify S3 URIs, following the principle of "parse, don't validate"
* **S3Client**: A client that provides convenient methods for interacting with S3 objects using type-safe S3Uri objects

Features
--------

* Type-safe S3 URI parsing and manipulation
* Convenient file operations (upload, download, open)
* Directory operations with concurrent uploads
* Support for both direct S3 access and local file operations
* Built on top of boto3 and fsspec

Quick Example
-------------

.. code-block:: python

   from conflux_s3_utils import S3Client, S3Uri

   # Initialize client
   s3 = S3Client()

   # Parse and manipulate S3 URIs
   base_uri = S3Uri.from_str("s3://my-bucket/data")
   file_uri = base_uri / "subfolder" / "file.txt"

   # Check if file exists
   if s3.object_exists(file_uri):
       # Download file
       with s3.open(file_uri, mode="rb") as f:
           data = f.read()

Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`
