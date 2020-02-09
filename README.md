```
                _____     _       _____     _
 _____ _ _ _   |  _  |_ _| |_ ___|  _  |_ _| |_
|     | | | |  |     | | |  _| . |   __| | | . |
|_|_|_|_____|  |__|__|___|_| |___|__|  |___|___|
```
<img src="https://www.mattseabrook.net/section31/images/tux.png" height=100px><img src="https://www.mattseabrook.net/section31/images/mediawiki.png" height=100px>

This software is a publishing automation tool suite for the MediaWiki software engine. It's primary features are allowing you to use whatever Text Editor/IDE you want, while updating the documentation in real-time every time you save your work in the local editor/documentation tool-chain of your choice. It also runs as a service quietly in the background- monitoring any type of repository for changes to documentation, and pushes the update automatically to your MediaWiki site.

Also included in this `README.md` file are recommended CSS & JavaScript customizations that can be applied to your MediaWiki instance, that will allow you to author and maintain your sources in regular mixed `Markdown` and `HTML`.  You will be able to have syntax highlighting without using any extensions, load resources from a CDN, and keep your sources decoupled in a way that will allow you to switch platforms and stop using this tool at any time.

Combined with this tool you will achieve a MediaWiki setup that is professional in appearance, while simultaneously giving you the leverage to implement whatever editing and IT Admin solutions work best on your end. Personally I use this tool in conjunction with MediaWiki to have my own customized version of an "Evernote"-type software that allows me to quickly take private notes, but also design elaborate Single Page Applications and share them both pubically and privately. I've also used it to stage documentation in a Technical Writing sandbox environment, and to automatically update customer facing Knowledge Bases when commits are made to a documentation repository.

## Table-of-Contents

<ol>
    <li><a href="#linux-environments)">LINUX Environments</a>
        <OL TYPE="a">
            <LI><a href="#pre-requisites">Pre-requisites</a>
            <LI><a href="#install">Install</a>
                <OL TYPE="a">
                    <LI><a href="#mwautopub">.mwautopub</a>
                    <LI><a href="#inotifywait">inotifywait</a>
                    <LI><a href="#cmark">cmark</a>
                </OL>
        </OL>
    </li>
    <li><a href="#usage">Usage</a>
        <OL TYPE="a">
            <LI><a href="#monitor">Monitor</a>
                <OL TYPE="a">
                    <LI><a href="#documentation-repo">Documentation Repo</a>
                </OL>
            <LI><a href="#help">Help</a>
            <LI><a href="#version">Version</a>
        </OL>
    </li>
    <li><a href="#logging">Logging</a>
    </li>
    <li><a href="#troubleshooting">Troubleshooting</a>
        <OL TYPE="a">
            <LI><a href="#known-issues">Known Issues</a>
            <LI><a href="#connectivity">Connectivity</a>
            <LI><a href="#developer-notes">Developer Notes</a>
        </OL>
    </li>
    </li>
    <li><a href="#mediawiki-enhancements">MediaWiki Enhancements</a>
    </li>
    <li><a href="#todo">TODO</a>
    </li>
</ol>

## LINUX Environments

### Pre-requisites:

- bash 4.3 (Support for associative arrays & declare/local -n)
- sed
- jq
- curl
- inotifywait
- cmark (C Reference implementation)

### Install

Clone or download this repository to your LINUX environment. Any such distribution and Mac OS X should be fully supported at this time (Windows coming soon.) Once you have a copy of this repo, run the following commands to install this software:

```bash
#!/bin/bash

# Create the configuration file
touch ~/.mwautopub

# Install mwautopub somewhere in the $PATH
cp mwautopub /usr/local/bin

# Create the log directory and set permissions
sudo mkdir /var/log/mwautopub && chown $UID:$UID /var/log/mwautopub/
```

This file needs to be in the user's $HOME directory.

#### .mwautopub

Here is a sample configuration file to get you started. It contains only the bare minimum for a Technical Writer to get started using the tool to publish content. Be sure to check out the other features of `mwautpub` below, such as logging, proxy server support, debug/verbosity options, etc.

```bash
username="Tex Murphy"
password="p@ssw0rd"
site="https://www.myhost.com/mediawiki/api.php"
```

#### inotifywait

If your Linux distribution didn't come with the `inotify-tools`, then they can be installed by running the following command:

```bash
#!/bin/bash

sudo apt-get install -y inotify-tools
```

#### cmark

`mwautopub` is configured to use Github's branch of cmark named `cmark-gfm`, which offers all of the extensions for Markdown to use `<table>` in your documentation. If you want to use a different version of the CommonMark specification, search the bash shell source file for `cmark-gfm` and change it accordingly.

This can be installed quickly by running the following in your terminal:

```bash
#!/bin/bash
 
sudo apt-get install -y cmark-gfm
```

## Usage

From the terminal you can run `mwautopub` without arguments to verify your installation, and that it's available in the `$PATH`:

```bash
#!/bin/bash

mwautopub
```

The output should be similar the following:

```text
                _____     _       _____     _
 _____ _ _ _   |  _  |_ _| |_ ___|  _  |_ _| |_
|     | | | |  |     | | |  _| . |   __| | | . |
|_|_|_|_____|  |__|__|___|_| |___|__|  |___|___|
Publishing/Editing automation tools for MediaWiki
v.1.3 - 02/05/2020, Authored by: Matthew Seabrook ( info@mattseabrook.net )

STATUS: OFFLINE

user@host: $
```

### Monitor

Usage: `mwautopub -m /path/to/documentation/repo/`

Watches the path supplied at run-time recursively for changes to any `Markdown` or `html` file and proceeds to perform the following workflow to automatically update your MediaWiki pages/articles:

<img src="https://www.mattseabrook.net/section31/images/mwautopub.png">

Care has been taken to avoid multiple CLOSE_WRITE events that happen in under 1 second. Multiple popular commercial and open source documentation tools do perform multiple CLOSE_WRITE events when saving and closing a file, and for this reason I have designed `mwautopub` to ignore multiple events in the same second of time.

#### Documentation Repo

Our example documentation repository in the image below is a folder located somewhere in our infrastructure. The folders inside the `repo` folder are git repos that contain the individual assets that ultimately comprise a wiki article/page, such as `Markdown`, `HTML`, `CSS`, `JavaScript`, and image data. The folder names correspond 1:1 to MediaWiki page URLs, and are used as the title parameter when POSTing the updated article to the MediaWiki API (*this limitation will be addressed in the next version.*)

<img src="https://www.mattseabrook.net/section31/images/docrepo.png">

**Custom settings per document**

In a future version there will be support for an `.mwautopub` configuration file in each folder or repo, that will expose the options better for watch file to watch, what the source format should be, the name/URL of the document to update, etc. 

**Stop the monitoring service**

The service will either stop when you log out of your session, or you can stop it at any time by running:

```bash
#!/bin/bash

mwautopub service stop
```

Similarly you can check the status of the service at any time by entering `mwautopub service status` or simply just `mwautopub` will always display it's status as either ONLINE or OFFLINE.

### Help

Display usage information about the available command-line arguments and how to invoke the various features of `mwautopub`.

### Version

Display version information about `mwautopub`

## Logging

`mwautopub` features a robust logging system! For convenience, and ease-of-use by both end-users and developers, separate log files are generated for *File System, MediaWiki Login, HTTPS,* and *Article Edit* events.

All actions are logged to three files in `/var/log/mwautopub/` folder:

|   File Name   | Description                                      |
| :-----------: | ------------------------------------------------ |
| mwautopub.log | General run-time error information               |
|  mwlogin.log  | Audit-trail & errors related to MediaWiki logins |
|  mwedits.log  | Documentation related activities                 |

### Exit Codes

Custom exit codes are defined in the range starting at 131, as per the [Advanced Bash-Scripting Guide](http://tldp.org/LDP/abs/html/index.html) and ```usr/include/sysexits.h``` from the **C** language (as a best practice.)

**Custom Exit codes**:

| Exit Code |     Type      | Description                                                   |
| :-------: | :-----------: | ------------------------------------------------------------- |
|    131    |  File System  | The **log** directory doesn't exist.                          |
|    132    |  File System  | Insufficient write-privilege for the **log** directory.       |
|    133    |  File System  | ```.mwautopub``` does not exist in the user's home directory. |
|    134    | Configuration | Missing or null parameter values in the configuration file.   |
|    135    |     Login     | MediaWiki login failed with the supplied credentials.         |

## Troubleshooting

This section contains valuable information for End-users, Administrators, and Developers!

### Known Issues

This is the current *Known Issues* list:

- double-quotes "" are required in the config file
- Space characters in the MediaWiki password field will fail
- Comments (#) are not supported in the ```.mwautopub``` config file

### Connectivity issues

Check your firewall of course, but beyond that I've implemented a couple of features to help troubleshoot networking issues that may prevent ```mwautopub``` from functioning properly:

1. ```cURL``` output has been redirected to ```/var/run/user/$UID/mwautopub/http.log``` by default. The verbosity of this log can be increased by adding ```verbosehttplogs="enabled"``` into the ```.mwautopub``` configuration file in your home directory.

2. Proxy server support is built in if you would like to use a tool such as Fiddler to view the HTTPS traffic, headers, query parameters, and body. Enable this feature by adding the following into the `.mwautopub` configuration file in your home directory:

```bash
proxy="192.168.0.1:8888"
```

### Developer Notes

- Lower-case variables in the shell source are 1:1 to parameter key names in the `.mwautopub`  config file
- Max user watches are 8192, but can be increased by writing to **/proc/sys/fs/inotify/max_user_watches.** Writers and Content Editors should come nowhere near this limitation, but if you carelessly point `mwautopub --monitor /path/` to a location containing thousands of folders and files, you will quickly run into operating system limitations.

## Maintenance

This tool assumes that your source documentation is being stored/backed up in some sort of repository, regardless of whether or not it's `git` or a `*.zip` file containing a folder of hundreds of documents. It will work well for you no matter what your process is, but the number of edits to individual articles on the MediaWiki system will get out of control with the use of any bot programs or automation tools. 

Acting on the premise that it is more desirable to be using Visual Studio Code as your modern editor, and have background processes manage the storage and publishing of your articles automatically, it is recommend to create a `cron` job or some such thing to schedule MediaWiki's built-in PHP script to remove **all** old revisions.

From the base directory of your MediaWiki deployment:

```bash
#!/bin/bash

php maintenance/deleteOldRevisions.php --delete
```

Here's how to automate this with `cron`. In the example below, replace `$MWPATH` with the full path to your MediaWiki instance.

```bash
#!/bin/bash

(crontab -l ; echo "00 09 * * 1-5 php $MWPATH/maintenance/deleteOldRevisions.php --delete") | crontab
```

## MediaWiki Enhancements

The following enhancements/customizations are recommended on a default "Hello World" installation of MediaWiki on your server. They will produce a document that looks like the image below, out of the box, without having to work with `PHP` (outside of enabling a few features in `LocalSettings.php`), the awful `Markup` syntax, or have any knowledge of the antiquated MediaWiki system.

<img src="https://www.mattseabrook.net/section31/images/docsample.gif">

### LocalSettings.php

The suggested changes to `LocalSettings.php` are described in the comments of the example below:

```php
# Load highlight.js and CSS from CDN, and custom.css/custom.js from your own server
$wgHooks['BeforePageDisplay'][] = function (OutputPage &$out, Skin &$skin) {
    $out->addScriptFile('https://cdnjs.cloudflare.com/ajax/libs/highlight.js/9.18.1/highlight.min.js');
    $out->addStyle('https://cdnjs.cloudflare.com/ajax/libs/highlight.js/9.18.1/styles/tomorrow-night.min.css');
    $out->addScriptFile('https://www.mydomain.com/mywiki/customization/custom.js');
    $out->addStyle('https://www.mydomain.com/mywiki/customization/custom.css');

};

# Allow <img> tags in pages/articles
$wgAllowImageTag = true;
```

### CSS

Feel free to change the fonts, they are being supplied currently from Google Fonts. Be sure this CSS loads on each page as per `LocalSettings.php`.

```css
/* custom.css */

@import url('https://fonts.googleapis.com/css?family=Source+Code+Pro:300|Open+Sans|Roboto&display=swap');

#firstHeading,
.mw-headline {
    font-family: 'Roboto', sans-serif;
}

ul,
p {
    font-family: 'Open Sans', sans-serif;
}

pre {
    padding: 0;
    border: none;
}

code {
    font-family: 'Source Code Pro', monospace;
}

.toctogglespan,
.toclevel-4 {
    display: none;
}

div#p-navigation ul {
    list-style-type: none;
    list-style-image: none;
}

#p-navigation {
    margin-left: -15px !important;
    font-size: 0.8em;
}

#content {
    margin-left: 13em;
    border: 1px solid #ccc;
    border-right: none;
}

#p-namespaces {
    background-image: none;
}

#mw-panel {
    position: fixed;
    top: 0;
}

#p-logo {
    margin-left: 1em;
}
```

### JavaScript

The following JavaScript loaded on each page in the front-end of the browser will move the **Table-of-Contents** into the left-side navigation and make it "sticky."

More importantly it fixes the issue with nested `<code>` tags not being rendered inside `<pre>` tags on the MediaWiki platform, which enables easy usage of Highlight.js via CDN.

```javascript
// custom.js

// MediaWiki front-end tweaks, (c) 2020 Matthew Seabrook (info@mattseabrook.net)

document.addEventListener('DOMContentLoaded', (event) => {
    const content = document.getElementById("mw-content-text")

    //
    // Fix all of the pre tags by repairing the nested <code> tag
    //
    for (const pre of content.getElementsByTagName("pre")) {
        const str = pre.innerHTML

        // Generate a new fixed <code> tag
        let array = str.split("&gt;"),
            code = array[0].replace(/&lt;/i, '<') + ">"

        // Inject the new <code> tag with the original proper <pre> text
        pre.innerHTML = code + str.substring(array[0].length + 4).slice(0, -14)

        // Initialize hljs on the <code> tag inside this <pre> tag
        hljs.highlightBlock(pre.getElementsByTagName("code")[0])
    }

    //
    // Table of Contents
    //
    const toc = document.getElementById("toc"),
        portal = document.getElementsByClassName("portal")[0]

    if (toc) {
        toc.parentNode.removeChild(toc)

        portal.innerHTML = toc.innerHTML
    } else {
        portal.style.display = "none";
    }
})
```

## TODO

- Send a custom `User-Agent` header identifying this custom software tool
- Remove the requirement for quotes "" around the value in the config file key/value pairs
- Implement more logging and error codes
- There is a `<table>` error similar to the nested `<code>` and `<pre>` tag issue, created by MediaWiki's sanitization/input engine. I need to write some JavaScript to fix this in the front-end, like I did for the color-highlighted syntax.
- Add more CSS classes for the `<table>`. Currently there is no border or row-shading.