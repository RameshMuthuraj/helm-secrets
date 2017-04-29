## Plugin for secrets management using Mozilla SOPS as backend

First internal version of plugin used pure pgp and whole secret file was encrypted as one.

Current version of plugin using Golang sops as backend which could be integrated in future into Helm itself, but currently it is only shell wrapper.

What kind of problems this plugins solves:
* Simple replacable layer integrated with helm command for encrypt, decrypt, view secrets files stored in any place. Currently using SOPS as backend.
* [Support for YAML/JSON structures encryption - Helm YAML secrets files](https://github.com/mozilla/sops#important-information-on-types)
* [Encryption per value where visual Diff should work even on encrypted files](https://github.com/mozilla/sops/blob/master/example.yaml)
* [On the fly decryption for git diff](https://github.com/mozilla/sops#showing-diffs-in-cleartext-in-git)
* On the fly decryption and cleanup for helm install/upgrade with this plugin helm bash command wrapper
* [Multiple key managment solutions like pgp and AWS KMS at same time](https://github.com/mozilla/sops#using-sops-yaml-conf-to-select-kms-pgp-for-new-files)
* [Simple addind/removing keys](https://github.com/mozilla/sops#adding-and-removing-keys)
* [With AWS KMS permissions managment for keys](https://aws.amazon.com/kms/)
* [Secrets files directory tree seperation with recursive .sops.yaml files search](https://github.com/mozilla/sops#using-sops-yaml-conf-to-select-kms-pgp-for-new-files)
* [Extracting sub elements from encrypted file structure](https://github.com/mozilla/sops#extract-a-sub-part-of-a-document-tree)
* [Encrypt only part of file if needed](https://github.com/mozilla/sops#encrypting-only-parts-of-a-file). [Example encrypted file](https://github.com/mozilla/sops/blob/master/example.yaml)

### Usage
```
$ helm secrets help
```
#### Available commands:
```
  enc           Encrypt chart secrets file
  dec           Decrypt chart secrets file
  dec-deps      Decrypt chart's dependecies' secrets files
  view          Print chart secrets decrypted
  edit          Edit chart secrets and ecrypt at the end
```
Any of this command have it's own help

#### SOPS as alternative usage in shell
As alternative you can use sops for example for edit just type
```
sops <SECRET_FILE_PATH>
```
Mozilla sops official [usage page](https://github.com/mozilla/sops#id2)

### Install

#### SOPS install
Just install plugin and sops will be installed using hook when helm > 2.3.x

You can always install manually for MacOS:
```
brew install sops
```
For Linux RPM or DEB, sops is available here: [Dist Packages](https://go.mozilla.org/sops/dist/)

#### SOPS git diff
Git config part is installed with plugin but for fully functional work need ```.gitattributes``` file inside root directory of charts repo with content
```
*.yaml diff=sopsdiffer
```
More info on [sops page](https://github.com/mozilla/sops#showing-diffs-in-cleartext-in-git)

#### Using Helm plugin manager (> 2.3.x)

```
helm plugin install https://github.com/futuresimple/helm-secrets
```

#### Pre Helm 2.3.0 Installation
Get a release tarball from the [releases](https://github.com/futuresimple/helm-secrets/releases) page.

Unpack the tarball in your helm plugins directory (```$(helm home)/plugins```).

For example:
```
curl -L $TARBALL_URL | tar -C $(helm home)/plugins -xzv
```

### Tips

#### Prevent commiting decrypted files to git
If you like to secure situation when decrypted file is committed by mistake to git you can add your secrets.yaml.dec files to you charts project .gitignore

As the second level of securing this situation is to add for example ```.sopscommithook``` file inside your charts repo local commit hook.
This will prevent commiting decrypted files without sops metadata.

```.sopscommithook``` content example:
```
#!/bin/sh

for FILE in $(git diff-index HEAD | grep <your vars dir> | grep "secrets.y" | cut -f2 -d$'\t'); do
    if file "$FILE" | grep -q -C10000 "sops:" | grep -q "version:"
    then
        echo "!!!!! $FILE" 'File is not encrypted !!!!!'
        echo "Run: helm secrets enc <file path>"
        exit 1
    fi
done
exit
```


### Real life use cases/examples


