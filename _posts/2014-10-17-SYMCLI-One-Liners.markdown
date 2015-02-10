---
layout: post
title: SYMCLI Singles
date: 2014-10-17 20:41:07.000000000 -04:00
categories:
- Automation
- SYMCLI
- Symmetrix
tags: []
status: publish
type: post
published: true
meta:
  _edit_last: '1'
author:
  login: scummins
  email: sean@scummins.com
  display_name: Sean Cummins
  first_name: Sean
  last_name: Cummins
---


This post is simply a collection of UNIX/Linux shell one-liners for SYMCLI. Well, I guess they're not all one-liners, but they're all pretty brief -- to be pasted into a shell rather than executed from a file. I'll often share individual code snippets with people on an ad hoc basis, but I figured it would be more useful to just share everything in one post.

I typically run these in [SYMCLI offline mode]({{ site.url }}/offline-symcli-commands/) from a CentOS-based Docker container running in an Ubuntu VM, which in turn runs in VMware Fusion on my Mac. If there's any interest, I can put together a post with some info on the whole 'Dockerized Solutions Enabler' thing (all of this is totally unsupported, BTW -- hence the 'offline mode' bit).

These examples use environment variables in place of specific object names, so you can either set your environment variables accordingly and then copy/paste the code, or you can manually replace the variables with explicit object names.

For example:

~~~bash
export SID=1234
export POOL=EFD
export DEV=1CA3
export SG=ESX_Storage_Group
~~~

Please note that these examples generally apply to the VMAX and VMAX2 product family, which primarily refers to the VMAX 10k, 20k, and 40k models. I'll call out exceptions to this as needed.

Have a SYMCLI one-liner to share? Need help with a one-liner? Find a bug in one of mine? Drop me an email, or stop by the comment section below.

This software is provided under the [MIT License](http://opensource.org/licenses/MIT) "AS IS", without warranty of any kind.


# Regular Expressions
Regular expressions that work well with SYMCLI output.

## Matching Device Names

**Note**: VMAX^3 uses 5-character device names, so you'll want to replace `\{4\}` with `\{5\}`.



SymDev ID in first column:

~~~bash
^[0-9a-fA-F]\{4\}\b
~~~

SymDev ID with optional leading spaces:

~~~bash
^\s*[0-9a-fA-F]\{4\}\b
~~~

![screenshot1]({{ site.url }}/assets/ss_regex_devmatch.png)



## Matching Storage Group Types

To show just parent SGs:

~~~bash
 symsg -sid $SID list |grep -e "[\.X][\.X][P][\.DSB]"
~~~

To show just child SGs:

~~~bash
symsg -sid $SID list |grep -e "[\.X][\.X][C][\.DSB]"
~~~

To show non-cascaded SGs:

~~~bash
 symsg –sid $SID list |grep -e "[\.X][\.X][\.][\.DSB]"
~~~

To show cascaded SGs:

~~~bash
 symsg -sid xxx $SID |grep -e "[\.X][\.X][CP][\.DSB]"
~~~

![screenshot1]({{ site.url }}/assets/ss_regex_sgmatch.png)


## Add colons to WWNs
To covert between, say, SYMCLI's and Brocade's WWN formats.

I'd imagine there's a better way to do this, but it's beyond my regex capabilties at this point :) This should work with anything that outputs WWNs without colons; I just happen to be using a symaccess command in the example below.

~~~bash
symaccess -sid $SID list devinfo |  \
sed -e 's/\b\([0-9a-f]\{2\}\)\([0-9a-f]\{2\}\)\([0-9a-f]\{2\}\)\([0-9a-f]\{2\}\)\([0-9a-f]\{2\}\)\([0-9a-f]\{2\}\)\([0-9a-f]\{2\}\)\([0-9a-f]\{2\}\)\b/\1:\2:\3:\4:\5:\6:\7:\8/ig'
~~~

![screenshot1]({{ site.url }}/assets/ss_regex_wwn_transform.png)



# Reports

## Storage Group Membership Report
Produces a report with one row per Storage Group, showing all members of each Storage Group.

First, ensure that Full Storage Group names are being used:

~~~bash
export SYMCLI_FULL_NAME=1
~~~

### Option 1: Full CSV
So you can import into Excel and split rows into columns on comma delimeters. Both the SG name and all devices will then end up in separate cells.

~~~bash
for sg in `symsg -sid $SID list |egrep "0$" |awk {'print $1'}`; do
   echo -n $sg
   for dev in `symsg -sid $SID show $sg \
      | grep -e "^\s*[0-9a-fA-F]\{4\}\b" | awk {'print $1'}`; do
         echo -n ', '; echo -n $dev; done; echo; done
~~~

![screenshot1]({{ site.url }}/assets/ss_rpt_sg_membership_csv.png)


### Option 2: SG Delimited with a hash (#)
So you can import into Excel and split rows into columns on hash delimeters. In this case, the SG name will be in the first column, and the complete device list (comma separated) will be in the second column.

~~~bash
for sg in `symsg -sid $SID list |egrep "0$" |awk {'print $1'}`; do
   echo -n $sg; echo -n '#'; for dev in `symsg -sid $SID show $sg \
   |grep -e "^\s*[0-9a-fA-F]\{4\}\b" |awk {'print $1'}`; do \
      echo -n $dev; echo -n ', '; done; echo; done
~~~

![screenshot1]({{ site.url }}/assets/ss_rpt_sg_membership_hash.png)


## Show devices that are not in Storage Groups

~~~bash
diff <(symsg -sid $SID list -v | grep -e "^\s*[0-9a-fA-F]\{4\}\b" \
   | awk {'print $1'} |sort -u) <(symdev -sid $SID list -nomember \
   | grep -e "^\s*[0-9a-fA-F]\{4\}\b" | awk {'print $1'} |sort -u)
~~~

![screenshot1]({{ site.url }}/assets/ss_rpt_nosg.png)


## Show FA mappings for a particular Storage Group

~~~bash
for dev in `symsg -sid $SID show $SG |grep -e "^\s*[0-9a-fA-F]\{4\}\b" \
   | awk {'print $1'}`; do
      symdev -sid $SID list -dev $dev -multiport \
      |grep -Eow "\b([0-9]{2}[E-H]:[0-1])\b"; done \
| sort -u
~~~

![screenshot1]({{ site.url }}/assets/ss_rpt_fa_mappings.png)



## Show TDAT variance for all pools in an array

~~~bash
POOLS=($(symcfg -sid $SID list -thin -pool \
   | grep -e " T[SFEM][FA89][EDNS][EDB][IXM] " |awk {'print $1'}))

for POOL in $POOLS; do
   echo "--- POOL ${POOL} ---"
   for i in `symcfg -sid $SID show \
      -thin -pool $POOL | grep -e "\s\+[0-9A-F]\{4\}\s\+" \
      | awk {'print $5'} | sort -u`; do \
         echo -ne "${POOL}, #TDATs at ${i}% full: "; symcfg -sid $SID \
            show -thin -pool $POOL | grep -e "\s\+[0-9A-F]\{4\}\s\+" \
            | awk {'print $5'} | grep $i | wc -l
            done; done

~~~

![screenshot1]({{ site.url }}/assets/ss_rpt_tdat_variance.png)



## Show TDEVs that are not under FAST VP control

~~~bash
diff <(symdev -sid $SID list -fast | grep TDEV | awk {'print $1'} \
   | sort) <(symdev -sid ${SID} list -tdev -nomember | grep TDEV \
   | awk {'print $1'} | sort)
~~~

![screenshot1]({{ site.url }}/assets/ss_rpt_fastless_tdevs.png)



# XML One-Liners
The examples in this section all use [SYMCLI's XML output]({{ site.url }}/parsing-symclis-xml-output-with-python/). All code snippets require xmllint, which is included in the libxml2 package. It's probably already installed by default on your Linux distro; but if not, you can run `sudo [yum/zypper] install libxml2` to install it.

One annoying aspect about xmllint is the way it formats its output (wonky delimeters, which you could fix up with sed). A better alternative (IMHO) is [xmlstarlet](http://xmlstar.sourceforge.net), but this is not installed by default on most Linux distros, and appears to have been dropped from Fedora [EPEL 7]().

## Show all members of a given meta device


~~~bash
symdev -sid $SID show $DEV -out xml_e \
   | xmllint --format --xpath \
   ".//Device/Dev_Info[dev_name='$DEV']/../Meta/Meta_Device/dev_name" -
~~~

![screenshot1]({{ site.url }}/assets/ss_xml_meta_memberlist.png)



## Show all IGs with visibility to a given device

~~~bash
symaccess -sid $SID list devinfo -out xml_e \
   | xmllint --format --xpath \
   ".//Initiator_Group/Group_Info/Device[dev_name='$DEV']/../group_name" -
~~~

![screenshot1]({{ site.url }}/assets/ss_xml_ig_visibility.png)



## Show all SGs containing a given device

~~~bash
symsg -sid $SID list -v -output xml_e \
   | xmllint --format --xpath \
   ".//SG/DEVS_List/Device[dev_name='$DEV']/../../SG_Info/name" -
~~~

![screenshot1]({{ site.url }}/assets/ss_xml_sg_membership.png)
