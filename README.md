# bookstore

[![Documentation Status](https://readthedocs.org/projects/bookstore/badge/?version=latest)](https://bookstore.readthedocs.io/en/latest/?badge=latest)

This repository provides tooling and workflow recommendations for storing, scheduling, and publishing notebooks.

## Automatic Notebook Versioning

Every save of a notebook creates an immutable copy of the notebook on object storage.

To ease implementation, we'll rely on S3 as the object store, using [versioned buckets](https://docs.aws.amazon.com/AmazonS3/latest/dev/Versioning.html).

<!--

Include diagram for versioning

-->

## Storage Paths

All notebooks are archived to a single versioned S3 bucket with specific prefixes denoting the lifecycle of the notebook:

- `/workspace` - where users edit
- `/published` - public notebooks (to an organization)

Each notebook path is a namespace that an external service ties into the schedule. We archive off versions, keeping the path intact (until a user changes them).

| Prefix                                  | Intent                 |
| --------------------------------------- | ---------------------- |
| `/workspace/kylek/notebooks/mine.ipynb` | Notebook in “draft”    |
| `/published/kylek/notebooks/mine.ipynb` | Current published copy |

Scheduled notebooks will also be referred to by the notebook key, though we'll need to be able to surface version Ids as well.

## Transitioning to this Storage Plan

Since most people are on a regular filesystem, we'll start with writing to the `/workspace` prefix as Archival Storage (writing on save using a `post_save_hook` for a Jupyter contents manager).

## Configuration

```python
# jupyter config
# At ~/.jupyter/jupyter_notebook_config.py for user installs on macOS
# See https://jupyter.readthedocs.io/en/latest/projects/jupyter-directories.html for other places to plop this

from bookstore import BookstoreContentsArchiver
# from bookstore.bookstore_config import BookstoreS3Settings # uncomment if you want to pass class below

c.NotebookApp.contents_manager_class = BookstoreContentsArchiver

c.Bookstore.workspace_prefix = "/workspace/kylek/notebooks"
c.Bookstore.published_prefix = "/published/kylek/notebooks"

# Optional, in case you're using a different contents manager
# This defaults to notebook.services.contents.manager.ContentsManager
# c.bookstore.Archiver.underlying_contents_manager_class = ADifferentContentsManager

# c.Bookstore.storage_class = BookstoreS3Settings
# or 
c.Bookstore.storage_class = "bookstore.bookstore_config.BookstoreS3Settings"

c.BookstoreS3Settings.bucket = "<bucket-name>"

# Note: if bookstore is used from an EC2 instance with the right IAM role, you don't
# have to specify these
c.BookstoreS3Settings.access_key_id = "mykey" #<AWS Access Key ID / IAM Access Key ID>
c.BookstoreS3Settings.secret_access_key = "anotherkey"# <AWS Secret Access Key / IAM Secret Access Key>
```
