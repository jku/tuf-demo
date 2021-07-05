# tufctl

tufctl is a (Work-In-Progress) TUF repository management tool. It could be useful as a demo tool, manual testing tool and as part of testing infrastructure.

The core idea of tufctl is process where 
* Various "edit <role>" commands can be used to modify individual metadata
* "snapshot" command handles updating snapshot and timestamp after targets have changed
* During these commands the tool takes care of 
  * expiry updates
  * metadata version numbers
  * file name changes
  * signing (with all keys available)
* Signing can also be done explicitly with "sign" command
* "status" command can be used to print the state of the repository

## Repository initialization example:

    # initialize git (currently required for tufctl)
    git init .

    # Create root metadata
    tufctl edit root init

    # Add keys for the roles, put keys in various keyrings
    # (sign with keys offline1 and offline2)
    tufctl edit root add-key root offline1
    tufctl edit root add-key root offline2
    tufctl edit root set-threshold root 2
    tufctl edit root add-key snapshot online
    tufctl edit root add-key timestamp online
    tufctl edit root add-key targets dev

    # Create other top-level metadata (sign with keys online or dev)
    tufctl edit timestamp init
    tufctl edit snapshot init
    tufctl edit targets init

    # Update snapshot/timestamp contents (sign with key online)
    tufctl snapshot

    git commit -a -m "initial top-level metadata"

## Delegation example:

    # Add delegation (sign with key dev)
    tufctl edit targets add-delegation --path "files/*" role1
    tufctl edit targets add-key role1 dev2

    # Create the delegate targets role (sign with key dev2)
    tufctl edit role1 init

    # Update snapshot/timestamp contents (sign with key online)
    tufctl snapshot

    git commit -a -m "Delegation to role1"

## Target info update example:

    # Developer uploads a file (sign with key dev2)
    tufctl edit role1 add-target files/file1.txt file1.txt

    # repository verifies the upload (sign with key online)
    tufctl snapshot

    git commit -a -m "Add target 'files/file1.txt'"

## TODO

* Metadata API: Add add_key() to Targets or Delegations
* Metadata API: Adding a role to Delegations is easy to do wrong (e.g. duplicates): make it a ordereddict
* Comment: Most of the complexity is in the file handling:
  * as a demo it's nice to have the files visible...
  * but sqlite or something would almost certainly make the code simpler
* Finish the edit sub-commands
  * Support for all delegated roles features somehow
  * support removing things (delegations, keys, roles?)
* NEW verify process:
  * TODO the hash for last known root.json is stored in local config (TOFU?) and verified
    so that git pull can't just switch trusted root
* Rethink the secrets handling: it's not designed at all
  * having private keys in .tufctl makes demos easy but ... is not a great demonstration
  * at minimum we need to store [keyid -> privkey] pairs -- but a readable name would be good too
  * supporting metadata that is modified by others (not just this tool) makes this a lot harder
    as we can't trust on keys not changing delegator or delegates
  * to be able to sign we need to know which keyid signs which role [role -> keyid(s)]
    * figuring this out from metadata is possible but requires
      A) finding the delegator of the key
      B) checking if key is for the delegated role
    * we could cache [keyid -> delegator] and regen the cache on a "keys update" command
    * or we could just not support moving keys or signing more than one role per key?
  * is there a need to use existing keys? maybe not at least as "supported feature" 
* goal: run tufctl in Github action as the "repository server":
  * Does timestamp updates
  * accepts developer changes as PRs (this is a bit scary from sec perspective)
  * Does snapshot updates after those developer changes
  * this unfortunately requires potentially unsafe github pull_request_target
  * Two major blockers:
    * have to make sure we verify _all_ changes in the repo:
      * need to store trusted root somewhere outside of git
      * could e.g. test that "git ls-files" only contains json files
      * prevent changing/affecting workflows?
    * Proper private key handling needs to be implemented
* Is it possible to take private keys and reading/writing metadata out of TufCtl object?
  * otherwise making tufctl into a "library" doesn't make a lot of sense
  * priv keys could be just arguments
  * for edit/sign commands the metadata probably could be just arguments to TufCtl?
  * the tricky parts are "snapshot" and "status" commands: they currently need all _json files_ as arguments
