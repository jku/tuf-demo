# tuf-repo-test

Test [tufrepo](https://github.com/vmware-labs/repository-editor-for-tuf) + Github Actions as an automated TUF repository:
 * snapshot and timestamp keys are stored in Github Secrets
 * Timestamp updates happen as cron action
 * Snapshot updates happen as an action after every push (if needed)

In practice developer can edit and sign e.g. targets with appropriate keys, push their changes, and the automation will update the snapshot so the repository is valid.
 
