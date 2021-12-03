# TUF repository automation demo

This is a demo of [TUF](https://theupdateframework.io/) repository automation
using [tufrepo](https://github.com/vmware-labs/repository-editor-for-tuf) and
Github Actions:
 * snapshot and timestamp keys are stored in Github Secrets
 * Timestamp updates happen as cron action
 * Snapshot updates happen as an action after every push (if needed)

In practice developers can edit and sign targets with appropriate keys _without
having access to snapshot and timestamp keys_, push their changes, and the
automation will update the snapshot so the repository is valid.

TUF clients can access the repository deployed in
https://jku.github.io/tuf-demo:

```bash
# Trust-on-first-use: Download initial root metadata
mkdir /tmp/metadata/
curl -o /tmp/metadata/root.json https://jku.github.io/tuf-demo/metadata/1.root.json
```

```python
from tuf.ngclient import Updater

url = "https://jku.github.io/tuf-demo/"
updater = Updater(
    repository_dir="/tmp/metadata/",
    metadata_base_url=f"{url}/metadata/",
    target_dir="./",
    target_base_url=f"{url}/targets/"
)

# Download file1.txt securely
info = updater.get_targetinfo("file1.txt")
if info is None:
    print("file1.txt not found")
elif not updater.find_cached_target(info):
    updater.download_target(info)
```
