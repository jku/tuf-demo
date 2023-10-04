# TUF repository automation demo

This is a live [TUF](https://theupdateframework.io/) repository, intended for
testing and demonstrations. It's
maintained with  [TUF-on-CI](https://github.com/theupdateframework/tuf-on-ci).

TUF clients can access the repository in https://jku.github.io/tuf-demo:

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
