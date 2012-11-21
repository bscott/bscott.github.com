---
layout: post
title: Chef 11 - Encrypted Databags Announcement
commentIssueId: 1
---

{{ page.title }}
================
<p class="meta">20 Nov 2012 - Los Angeles</p>

* The format of encrypted data bag items is changing in a breaking way
* The next 10.x release of Chef will include backwards compatibility with Chef 11 encrypted data bag items.
* Chef 10.16.2 and lower will not be able to read encrypted data bag items created with Chef 11.
* This fixes a bug where creating encrypted data bag items on one version of ruby and reading them with another could cause errors or even return incorrect data.
* http://tickets.opscode.com/browse/CHEF-3393
* http://tickets.opscode.com/browse/CHEF-3392
* http://tickets.opscode.com/browse/CHEF-3480


## Background
Encrypted data bag items support nested structures within an encrypted value, for example:

"id": "my-encrypted-item",
"value": {
  "password1": "opensesame",
  "password2": "supersecret"
}

To make this work, Chef first converts the value-to-encrypt to YAML and then encrypts it. To read the item, Chef decrypts the value and then parses the YAML.

In ruby 1.8-1.9.2, the default YAML implementation is called "syck". Unfortunately, syck is buggy and doesn't follow the YAML standard. In ruby 1.9.3+, the default YAML implementation is a wrapper around libyaml, called "psych". These two implementations are not entirely compatible. Depending on which implementation was used to create and which to read, we've seen cases where reading the YAML from the encrypted data bag item would cause errors, or worse, incorrect data.

For now, users can workaround the issue by explicitly choosing a YAML implementation (ruby 1.9 has both):

  YAML::ENGINE.yamler = 'syck'
or
  YAML::ENGINE.yamler = 'psych'

Unfortunately, this workaround will not be viable for very long, because ruby 2.0 removes the "syck" implementation completely.

## Chef 11 Changes
In Chef 11 we've changed the format of encrypted data bag items to use JSON to serialize data structures. This will allow us to support 1.8, 1.9 and 2.0 simultaneously. In addition, each encrypted value has a metadata wrapper containing an initialization vector for the block cipher, a format version, and the cipher used for the encryption. For example, after encryption, an encrypted value looks like this:

{"encrypted_data"=>"4UfhFPRtPOw3sh5ktdP0DNLFf6OgtvpZ0uwQdJKhuY0=\n",
 "iv"=>"ixkE67xS1w0tfFT4/dW4Kw==\n",
 "version"=>1,
 "cipher"=>"aes-256-cbc"}

This change allows us to cleanly introduce new formats in the future (there are no plans to do so, however), and potentially also add support for user-selectable ciphers. One consequence of this new format is that the "iv", "version", "cipher", and "encrypted_data" keys will be indexed for search, so keep this in mind if you use search with encrypted data bag items.

Also note that use of a random iv is new in Chef 11, which makes encrypted data bag items more resilient to certain forms of cryptanalysis.

## Compatibility and Upgrading
This new format is not compatible with the current format. When attempting to read a Chef 11 format encrypted data bag item with an incompatible version of Chef, you will see an error like this:

NoMethodError: undefined method `unpack' for #<Hash:0x007ff5b264e1f0>

To ease the task of upgrading, we've added compatibility with the new format to the 10-stable branch of Chef; this will be shipped with the next 10.x release (10.16.4 or 10.18.0, depending).

When upgrading, users of encrypted data bag items should do the following:
1. Upgrade all nodes that read encrypted data bag items to at least 10.16.4/10.18.0.
2. Upgrade chef on the workstation where you create/update encrypted data bag items to 11.0

If you want to take advantage of the security improvements in the new format immediately, re-upload your encrypted data bag items. Note that Chef 11 can read Chef 10.x format encrypted data bag items, so this step is optional.

I'll be coming out with a knife plugin to essentially downloads/decrypts/re-encrypts/uploads each encrypted databag item in a databag for easy migration to Chef 11's new format. Which is now here: https://github.com/leftathome/knife-databag-upgrade (Try at your own risk)