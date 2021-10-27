# TUF repository automation demo

This is a demo of [TUF](https://theupdateframework.io/) repository automation using [tufrepo](https://github.com/vmware-labs/repository-editor-for-tuf) and Github Actions:
 * snapshot and timestamp keys are stored in Github Secrets
 * Timestamp updates happen as cron action
 * Snapshot updates happen as an action after every push (if needed)

In practice developers can edit and sign targets with appropriate keys _without having access to snapshot and timestamp keys_, push their changes, and the automation will update the snapshot so the repository is valid.

TUF clients can access the repository right from GitHub using the raw content URLs:

```bash
# Trust-on-first-use: Download initial root metadata
mkdir /tmp/metadata/
curl -o /tmp/metadata/root.json https://raw.githubusercontent.com/jku/tuf-repo-test/master/metadata/1.root.json
```

```python
from tuf.ngclient import Updater

url = "https://raw.githubusercontent.com/jku/tuf-repo-test/master"
updater = Updater(
    repository_dir="/tmp/metadata/",
    metadata_base_url=f"{url}/metadata/",
    target_base_url=f"{url}/targets/"
)
updater.refresh()

# /tmp/metadata now has current top-level metadata
```
