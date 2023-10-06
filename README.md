# TUF repository demo

This is a live [TUF](https://theupdateframework.io/) repository, intended for
testing and demonstrations. It's
maintained with  [TUF-on-CI](https://github.com/theupdateframework/tuf-on-ci).

The goals of this project are:
* Maintain a long term repository with daily changes that the TUF community can use as a
  live example and as a remote repository for client testing that requires one
* Allow anyone in the TUF community to maintain artifacts in the repository for those purposes
* Act as the interactive testing platform for TUF-on-CI (the repository maintenance tool)

Being a secure delivery mechanism for any specific artifacts is not a goal: this is a demo. If you have questions, the
[TUF-on-CI slack](https://cloud-native.slack.com/archives/C04SHK2DPK9) is a good place.

### Becoming a signer

We'll try to offer anyone in the TUF community the ability to add and modify artifacts in this repository. 

The process is not formalized yet but if you'd like to become a signer please open an issue and we'll
go from there:
* As a general rule you can expect to get your own delegation where you can add artifacts as you want
  (relatively small ones for now please: the system only stores artifacts in git at the moment)
* A Yubikey or similar hardware key is required as a signing method (Sigstore may be an option in future)
* You will need to install and use
  [tuf-on-ci-sign](https://github.com/theupdateframework/tuf-on-ci/blob/main/docs/SIGNER-MANUAL.md) 

### Using the repository with a TUF client

The repository is published at `https://jku.github.io/tuf-demo`. Metadata and artifacts are in subdirectories
`metadata` and `targets` respectively (so as an example current timestamp metadata is found in 
https://jku.github.io/tuf-demo/metadata/timestamp.json). 

A Python example client is provided below but any TUF client should work.

<details>
  <summary>Example Python client</summary>

```python
# Example client for https://jku.github.io/tuf-demo repository
# Usage example:  ./client.py file1.txt

import os, requests, sys
from tuf.ngclient import Updater

url = "https://jku.github.io/tuf-demo/"
metadata_dir = "/tmp/tuf-demo/"

if len (sys.argv) != 2:
    sys.exit(f"Usage:  {sys.argv[0]} <targetpath>")

# Trust-on-first-use: Download initial root metadata if it's not available
if not os.path.exists(f"{metadata_dir}/root.json"):
    os.makedirs(metadata_dir, exist_ok=True)
    with open(f"{metadata_dir}/root.json", "wb") as f:
        f.write(requests.get(f"{url}/metadata/1.root.json").content)

# Download target securely using python-tuf
updater = Updater(
    metadata_dir=metadata_dir,
    metadata_base_url=f"{url}/metadata/",
    target_dir="./",
    target_base_url=f"{url}/targets/"
)
info = updater.get_targetinfo(sys.argv[1])
if not info:
    print(f"'{sys.argv[1]}' not found")
    sys.exit()

path = updater.find_cached_target(info)
if path:
    print(f"'{path}' is already up-to-date")
    sys.exit()

path = updater.download_target(info)
print(f"Downloaded '{path}'")
```
</details>
