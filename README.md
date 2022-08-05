# TUF repository automation demo

This is a demo of [TUF](https://theupdateframework.io/) repository automation
using [repository-editor-for-tuf](https://github.com/vmware-labs/repository-editor-for-tuf) and
Github Actions:
 * snapshot and timestamp keys are stored in Github Secrets
 * Timestamp updates happen as cron action
 * Snapshot updates happen as an action after every push (if needed)

In practice developers can edit and sign targets with appropriate keys _without
having access to snapshot and timestamp keys_, push their changes, and the
automation will update the snapshot so the repository is valid.

TUF clients can access the repository deployed in
https://jku.github.io/tuf-demo:

```python
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

# Download file1.txt securely using python-tuf
updater = Updater(
    metadata_dir=metadata_dir,
    metadata_base_url=f"{url}/metadata/",
    target_dir="./",
    target_base_url=f"{url}/targets/"
)
info = updater.get_targetinfo(sys.argv[1])
if info is None:
    print(f"'{sys.argv[1]}' not found")
elif updater.find_cached_target(info):
    print(f"'{sys.argv[1]}' is already up-to-date")
else:
    updater.download_target(info)
    print(f"'{sys.argv[1]}' downloaded")
```
