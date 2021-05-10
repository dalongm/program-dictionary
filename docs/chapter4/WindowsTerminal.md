# Windows Terminal

[TOC]

## ssh: “perl: warning: Setting locale failed”

According to [this page](https://7dc.org/index.php/2018/12/04/centos-7-fix-for-perl-warning-please-check-that-your-locale-settings-are-supported-and-installed-on-your-system/), you can just add the following lines to **/etc/environment** file:

```
LC_ALL="en_US.UTF-8"
LC_CTYPE="en_US.UTF-8"
LANGUAGE="en_US.UTF-8"
```

