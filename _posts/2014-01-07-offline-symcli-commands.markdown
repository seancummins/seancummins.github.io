---
layout: post
title: Offline SYMCLI Commands
date: 2014-01-07 21:21:05.000000000 -05:00
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


One of the little-known features of  SYMCLI is its ability to interrogate an offline SYMAPI database. This can be useful for safely running query scripts without the risk of affecting a production array. Or if you're an EMC partner or employee, you can have customers send you a SYMAPI database so you can extract information about their array(s) while you're offsite. Many of our internal and partner tools -- things like Tier Advisor, BCSD, and SymmMerge -- also use SYMAPI databases to import array configuration data.

First, you of course need to have a SYMAPI database. There are a several ways to get it:

## SYMAPI DB from a live management host

Typically, you will pull a SYMAPI DB from a host running Solutions Enabler with access to Symmetrix Gatekeeper devices (i.e. a SYMCLI or Unisphere for VMAX management server). Before copying the SYMAPI DB, I'd recommend refreshing its contents with the following commands; this will ensure that the database reflects the current array configuration.

~~~bash
symcfg discover
symcfg sync -local
symcfg sync -vpdata
symcfg sync -fast
~~~

Now that the database has been updated, simply copy it to an offline location (e.g. your laptop). For Windows, the default SYMAPI DB path is `C:\Program Files\EMC\SYMAPI\db\symapi_db.bin`. For UNIX/Linux, you'll most likely find it at ``/usr/emc/API/symapi/db/symapi_db.bin`.

## T1 SYMAPI bins from STP Performance Data

If you happen to have STP (Symmetrix Trends & Performance) data, you most likely have a bunch of files that begin with T1 or T2 -- the files with a .btp or .ttp extension contain the performance data, and the files with a .bin extension are just SYMAPI databases.

## SYMAPI DB from array service processor

Finally, if you happen to have access to the VMAX Service Processor (i.e. you're a partner or employee with the appropriate credentials), you can obtain it there -- use the same instructions as for a regular SYMCLI management station.

# Using your offline SYMAPI database

In order to execute offline commands, you'll need to have Solutions Enabler installed. You can obtain this from [http://support.emc.com](support.emc.com).

Start a Command Prompt or Shell session, and set the following environment variables.

~~~bash
export SYMCLI_OFFLINE=1
export SYMCLI_DB_FILE=<path_to_SYMAPI_DB>
~~~

Run `symcli -def` to validate that your environment variables are set (they should be displayed in this command's output).

Now you can execute most "query" type commands against the offline database. There are some query commands that require online access, but you'll find that most can be run offline. Of course commands that would make changes to an array's configuration (e.g. provisioning devices) will only work online. You can also change almost any command's output to XML by adding the ``-output xml_e` flag -- this tends to be more reliable than screen scraping when programmatically parsing SYMCLI command output. In a follow-up post, I'll provide some examples of how to parse SYMCLI XML output using Python.

Here are a few commands to get you started:

~~~bash
# Detailed info on the arrays in the DB
symcfg list -v

# Info about thin pool capacity (optional –v for verbose)
symcfg list –thin –pool –detail –gb

# Physical disk info
symdisk list –dskgrp_summary

# Info about Symm directors (optional –v for verbose)
symcfg –dir all list –v

# List storage groups (optional –v for verbose)
symsg list

# List tier info (optional –v)
symtier –sid xxx list –offline

# List fast policies (optional –v)
symfast –sid xxx list –vp

# FAST Demand report by SG Association -- shows the potential for each SG to tier up based on assigned policies
symfast –sid xxx list –demand –assoc
~~~
