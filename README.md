# DRAFT reproducible-builds-verification-format

This document suggests a format to share results of verification builds.
Verification builds describe the process of additional creation of binary
artifacts to proof their reproducibility. The motivation is to make sure binary
distributors share what they claim. The scope are mainly distributions sharing
pre-compiled packages but may also include independent binary repositories.

The described format allows a generic way to compare results and also
comparability between distributions offering the same package.

Builders must sign the result JSON file with PGP, signify or both.

The result file has to be gziped.

## Results file skeleton

    {
    	"origin_uri": "",
    	"origin_name": "",
    	"results": [
    		{
    			"suite": "",
    			"component": "",
    			"target": "",
    			"name": "",
    			"version": "",
    			"cpe": "",
    			"status": "",
    			"artifacts": {
    				"buildlog_uri": "",
    				"diffoscope_html_uri": "",
    				"diffoscope_json_uri": "",
    				"binary_uri": ""
    			},
    			"build_date": 0,
    			"build_duration": 0
    		}
    	]
    }

## Field definition

This section describes the meaning of all fields of the result file. All fields
are mandatory unless marked as optional.

### `origin_uri` (string)

Binary source to compare to. In most cases the official binary download server
of a distribution. May also be a binary repository by a third party.

#### Example property

-   `http://ftp.us.debian.org/debian/`
-   `https://downloads.openwrt.org/`
-   `https://download.docker.com/`

### `origin_name` (string)

Descriptive name of the origin source used. This field may only contain ASCII
letters including the characters `-` and `_`. The value is used as a unique
distribution identifier in the database.

#### Example property

-   `debian`
-   `openwrt`
-   `docker`

### `results` (list)

Contains dictionaries with test results.

### `suite` (string)

The suite aka distribution branch identifier. This distinguishes between
available branches in which a binary is distributed by the same distribution.

#### Example property

-   `buster`
-   `19.07`
-   `rolling`

### `component` (string)

The component aka distribution subbranch, used when multiple repositories are
available per suite.

#### Example property

-   `main`
-   `core`
-   `community`
-   `iso`

### `target` (enum)

The used architecture of the binary. As distributions use different identifiers
for their architecture (Debian: amd64, Arch Linux: x86_64, ...) this property
can not be changed as desired to keep interpretability.

Additionally a common architecture name is not enough as neither a linux kernel
nor the gcc compiler is granted. A full target description is required. For
this purpose the rustc target list is suggested.

A complete list TBA.

#### Example property

-   `x86_64-unknown-linux-gnu`
-   `mipsel-unknown-linux-musl`
-   `aarch64-linux-android`

### `name` (string)

The distribution specific identifier to create the binary. This may not be the
name of the created binary itself but for instance a package name. This string
should not contain any version information.

#### Example property

-   `tmux`
-   `procd`
-   `firefox`

### `cpe` (string) [optional]

Search Common Platform Enumerations. The CPE of the binary if applicable. This
is mostly relevant for packages and may stay empty for other binary files like
iso images. The CPE string may only contain vendor and product identifier,
without any additional version information as the downstream version used may
does not match with upstream due to patches. CPE in version 2.3 should be used.
Offering this information allows interpretability between distributions.

#### Example property

-   `cpe:2.3:a:mozilla:firefox:*:*:*:*:*:*:*:*`
-   `cpe:2.3:a:haxx:curl:*:*:*:*:*:*:*:*`

### `status` (enum)

The status describes the test result of the binary. There is a predefined list
of statues to chose from.

All states are explained blow:

-   `reproducible`: recreated binary has the same checksum
-   `unreproducible`: binary compiles but results in a different checksum
-   `buildfail`: the creation was not successful and the build system failed
-   `notfound`: the sources required for the package are not available (anymore)
-   `timeout`: the build exceeded a specified time limit
-   `blocked`: the binary where actively excluded from testing
-   `notforus`: the binary is not compatible with the active target
-   `untested`: the binary has not yet been tested
-   `depwait`: the binary depends on an to be created other binary

### `artifacts` (dictionary)

Contains information in case the test did not result in reproducible to help
further evaluation. The allowed fields are described below.

### `buildlog_uri` (string) [optional]

URI to the buildlog created during the binary artifact creation.

### `diffoscope_html_uri`, `diffoscope_json_uri` (string)

URI to the diffoscope html and json output based on the origin file and the
rebuild binary.

### `binary_uri` (string) [optional]

URI to the created files to allow manual inspection.

### `build_date` (integer)

UNIX time stamp of the date and time the binary was created.

### `build_duration` (integer) [optional]

Duration of the build process for the binary artifact in seconds.
