Installation
============

Requirements
------------

* Python 3.12 or higher
* boto3 >= 1.0
* fsspec >= 2024

Installing from PyPI
--------------------

You can install Conflux S3 Utils using pip:

.. code-block:: bash

   pip install conflux-s3-utils

Installing from Source
----------------------

To install from source, clone the repository and install using pip:

.. code-block:: bash

   git clone https://github.com/conflux-xyz/conflux-s3-utils.git
   cd conflux-s3-utils
   pip install .

Development Installation
------------------------

For development, you can install with development dependencies:

.. code-block:: bash

   git clone https://github.com/conflux-xyz/conflux-s3-utils.git
   cd conflux-s3-utils
   pip install -e ".[dev]"

AWS Credentials
---------------

Since Conflux S3 Utils uses boto3 under the hood, you'll either need to configure your AWS credentials
for boto3, or you can construct your own instance of the boto3 client and pass it into the S3Client.

For more information, see the `boto3 credentials documentation <https://boto3.amazonaws.com/v1/documentation/api/latest/guide/credentials.html>`_.
