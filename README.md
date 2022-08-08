# TUF repository automation demo

This is a demo of [TUF](https://theupdateframework.io/) repository automation
using [repository-editor-for-tuf](https://github.com/vmware-labs/repository-editor-for-tuf) and
Github Actions:
 * snapshot and timestamp keys are stored in Github Secrets
 * Timestamp updates happen as cron action
 * Snapshot updates happen as an action after every push (if needed)
 * During the snapshot update, any available targets keys will be used to sign targets metadata

In practice developers can edit and sign targets with appropriate keys _without
having access to snapshot and timestamp keys_, push their changes, and the
automation will update the snapshot so the repository is valid.

TUF clients can access the repository deployed in
https://jku.github.io/tuf-demo:

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
