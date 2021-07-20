# TUF repository automation demo

This is a demo of [TUF](https://theupdateframework.io/) repository automation using [tufrepo](https://github.com/vmware-labs/repository-editor-for-tuf) and Github Actions:
 * snapshot and timestamp keys are stored in Github Secrets
 * Timestamp updates happen as cron action
 * Snapshot updates happen as an action after every push (if needed)

In practice developers can edit and sign targets with appropriate keys _without having access to snapshot and timestamp keys_, push their changes, and the automation will update the snapshot so the repository is valid.
