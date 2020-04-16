[Home](../index.md)

## Bash tips

![bash splash](res/bash-splash.png)

### Bash prerunner

Sources: 

* [Add the following after shebang to catch and avoid unwanted errors/side effects](https://twitter.com/nixcraft/status/1250480524872658946)
* [What is the difference between “#!/usr/bin/env bash” and “#!/usr/bin/bash”?
](https://stackoverflow.com/a/16365367)

```bash
#!/usr/bin/env bash
set -euo pipefail
```

### Replace text in a file with a unix style path

```bash
FIND='@PLACEHOLDER@'
REPLACE=`pwd` #replace is the text to be inserted

sed -i "s/${FIND}/${REPLACE}//\//\\\/}/g" targetfile.conf
```

### Exit the script if the previous command failed

```bash
[ $? -eq 0 ] || exit 1
```

### Perform a task and exit the script if the previous command failed

```bash
[ $? -eq 0 ] || ( <THE_TASK>; exit 1 ) || exit 1
```

### Get the directory containing the current script

```bash
DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" >/dev/null 2>&1 && pwd )"
```

from: https://stackoverflow.com/questions/59895/how-to-get-the-source-directory-of-a-bash-script-from-within-the-script-itself

### Normalise a directory name removing ".." in the path, etc.

```bash
NORMALISED="$( cd "${DIR}" >/dev/null 2>&1 && pwd )"
```

### Check pgp signatures

* Import public key using the key fingerprint
  * Eg from: https://www.gentoo.org/downloads/signatures/
* Verify the document against the signature

```bash

[ $(gpg --list-keys | grep ${KEY_SIG}) ] || gpg --keyserver hkps://keys.gentoo.org --recv-keys ${KEY_SIG}
gpg --verify ${FILE}.gpgsig ${FILE}

[ $? -eq 0 ] || exit 1
```
