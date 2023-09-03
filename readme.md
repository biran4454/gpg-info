
# How to use GPG - Cheatsheet

## Types of files
- .gpg - generic gpg file
- .sig - binary signature file, used to verify that a file is authentic
- .asc - armoured signature, so plaintext and readable

## Types of Key Identification
Any of the following can be used in place of `<key-ID>`
- Key fingerprint: SHA-1 / MD5 hash of the key, best option (albeit the longest)
- Long key-ID: last 8 bytes of the fingerprint, risky to use days as it is technically possible to create another key with an identical key-ID
- Short key-ID: last 4 bytes of the fingerprint, definitely risky and different keys with identical short key-IDs *have* been demonstrated possible

# Things needed
## To Verify a File
- Original file
- `<filename>.sig` or `<filename>.asc`
- **OR** Just `<filename>.gpg`, which contains the original file
- Public key (preferably trustworthy)

## To Create a Key
- Your name + email address
- A plan of what to use it for, and a timeframe
- A secure passcode / passphrase and a place to store it (preferably not your brain, depending on your threat model)
- Keyserver / Friends (optional, don't worry)

## To Sign a File
- A private key suitable for signing + the passcode / passphrase
- A file

## To Encrypt a File
- A private key suitable for encryption (and preferably signing) + the passcode / passphrase
- A file
- The public key of the recipient

# Key Management

## Obtaining Public Key
To get the public key, either:
- Copy + Paste key from website
- Download key from website
- Locate the key using the key ID or email
- Auto locate the key using the email address
- Search a keyserver / internet for the key

### Refresh Keys from Keyserver
1. `$gpg --refresh-keys`, optionally with `--keyserver` to select the server.

### Copy + Paste Public Key
1. `$vim <companyname>.gpg` to open a new key file
2. `[Ctrl+V]` to paste the public key
3. `:wq` to write the file and quit vim OR `:q!` to exit and try again
4. Go to Import Public Key

### Fetch Using Email
1. Obtain the company / user's email address
2. `$gpg --search-keys <email / key-ID>`

### Fetch Using Key ID
1. Obtain the key ID from the signature file
2. `$gpg --recv-keys <key-ID>`

### Auto Locate Key
1. Find the email address of the owner
2. `$gpg --auto-key-locate --locate-keys <email>` or `gpg --auto-key-locate nodefault,wkd --locate-keys <email>` to exclude searching the local keyring

### Import Public Key
`$gpg --import <companyname>.gpg` to import the new key

### Set Key Trust Level
1. `$gpg --list-keys` to show the new key and get the key hash
2. `$gpg --edit-key <keyhash> trust` to set the trust level
3. `<trust level>` OR `m` to abort and exit to main menu
4. `q` to quit

## Managing Keys
### Generate a Key Pair
1. `$gpg --full-generate-key`
2. `1` (or `4` if only for signing)
3. enter expiration time (or 0)
4. `y`
5. (enter name and email, and a comment so you don't get confused later)
6. `O`
7. Enter a secure passphrase / passcode. Ideally your private key should be kept securely so this shouldn't be necessary in an ideal world, but private keys are inevitably lost, accidentally uploaded, compromsed etc. and this gives peace of mind. Store the passphrase / passcode somewhere safe and inaccessable (locked KeePass database on a pendrive in a secure location, or a notebook in a secure location).
8. (wait for random generation, shouldn't take very long)
**Optional:**
9. Export with `$gpg --output <file>.keyring --export <key-ID>`
10. Give someone trusted your revocation certificate in case your private key is lost, or you are compromised
11. Tell everyone your (public) key hash and give everyone your public key and email, write it on your cards or post it on a permanent archive, and make sure people know how to find and verify it
12. Upload to a keyserver with `$gpg --send-key <key-ID>` (you can select a keyserver with the `--keyserver` option). Don't forget to verify your email address.
13. Have someone you know, and who other people trust, sign your public key

### List Private (Secret) Keys
`$gpg --list-secret-keys`

### Revoke a Key
1. If you have a revocation certificate, jump to step 4
2. `$gpg --output revoke.asc --gen-revoke <key-ID>`
3. `$gpg --import <revocation_cert>.asc` to mark that key as revoked
**If you have uploaded to a keyserver:**
4. `$gpg --keyserver <server> --search-keys <key-ID>`
5. `$gpg --keyserver <server> --send-keys <key-ID>`

### Delete a Key
`$gpg --delete-key <key-ID>`

### Export Public / Private Key
 `$gpg -a --export <key-ID> > <name>.asc` OR `$gpg -a --export-secret-key <key-ID> > <secret_name>.asc`

### Export Key to Keyring
Use this to manage your keys or to transfer it to someone else (or you could just export the key)
`$gpg --output <name>.keyring --export <key-ID>`

# Signing + Encryption
## Signing a File
### Integrated Signature
1. `$gpg --sign <file>` assuming the file is in cwd OR `$gpg -local-user 0x<key-ID> --sign <file>` to sign with a specific key
2. Enter the passphrase
3. The file will be `<file>.gpg`
### Detached Signature
1. `$gpg --detach-sig --sign <file>` assuming the file is in cwd OR `$gpg --detach-sig -local-user 0x<key-ID> --sign <file>` to sign with a specific key
2. Enter the passphrase
3. The file will be `<file>.sig`

## Signing a Public Key
1. Import the public key if you haven't already
2. `$gpg --list-keys <email>`
3. `$gpg --sign-key <key-ID>`
### Optional: upload the signed key
4. `$gpg --send-key <key-ID>` *NOTE: This will show others that you trust this key, however this can be a downside if you don't want others to know you have used this key*

## Signing + Encrypting a File
1. Ensure the file is in cwd
2. `$gpg --encrypt --sign -r <email / key-ID> <file>`. GPG will sign *then* encrypt the file.

# Verification + Decryption
## Verify Public Key
1. `$gpg --fingerprint <email>`
2. Phone / Zoom / Talk to the person IRL and ask what their key fingerprint is, and check they match
3. If they do match, you can probably improve the trust of the fingerprint (but never set it to Ultimate)
4. Optionally sign the key and upload to a keyserver, to show yourself and others that you trust this key to be authentic

## Verify File Signature
### With File + File.sig
1. Make sure `<file>` and `<file>.<sig|asc>` are in the cwd
2. Either Step 3 (lite mode) or Step 4 (heavy mode) or Step 5 (ultra lite)
3. `$gpgv <file>.<sig|asc>` OR `$gpgv <signature>.<sig|asc> <file>` (if the file is not named the same as the signature). 
*Important: gpgv does not check for revoked or expired keys, and assumes ALL keys in the keyring are trustworthy and valid*
4. `$gpg --verify <file>.<sig|asc>` OR `$gpg --verify <signature>.<sig|asc> <file>`
5. Check that it says "Good signature made"

### With File.gpg
*Note: If your file is encrypted, you will need to decrypt it instead*
1. Make sure `<file>.gpg` is in cwd
2. `$gpg --verify <file>.gpg`
3. Check that it says "Good signature made"
4. `$gpg --output <file> --decrypt <file>.gpg` (will also automatically verify)

## Decrypt File
1. `$gpg --output <file> --decrypt <file>.gpg`. This will automatically verify the decrypted file if it is signed and you have the public key
