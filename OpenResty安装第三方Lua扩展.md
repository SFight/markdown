# OpenResty安装第三方Lua扩展

## 一、推荐使用opm进行安装

```shell
opm -h


opm [options] command package...

Options:
    -h
    --help              Print this help.

    --install-dir=PATH  Install into the specified PATH directory instead of the system-wide
                        OpenResty installation tree containing this tool.

    --cwd               Install into the current working directory under ./resty_modules/
                        instead of the system-wide OpenResty installation tree containing
                        this tool.

Commands:
    build               Build from the current working directory a package tarball ready
                        for uploading to the server.

    clean ARGUMENT...   Do clean-up work. Currently the valid argument is "dist", which
                        cleans up the temporary files and directories created by the "build"
                        command.

    info PACKAGE...     Output the detailed information (or meta data) about the specified
                        packages.  Short package names like "lua-resty-lock" are acceptable.

    get PACKAGE...      Fetch and install the specified packages. Fully qualified package
                        names like "openresty/lua-resty-lock" are required. One can also
                        specify a version constraint like "=0.05" and ">=0.01".

    list                List all the installed packages. Both the package names and versions
                        are displayed.

    remove PACKAGE...   Remove (or uninstall) the specified packages. Short package names
                        like "lua-resty-lock" are acceptable.

    search QUERY...     Search on the server for packages matching the user queries in their
                        names or abstracts. Multiple queries can be specified and they must
                        fulfilled at the same time.

    server-build        Build a final package tarball ready for distribution on the server.
                        This command is usually used by the server to verify the uploaded
                        package tarball.

    update              Update all the installed packages to their latest version from
                        the server.

    upgrade PACKAGE...  Upgrade the packages specified by names to the latest version from
                        the server. Short package names like "lua-resty-lock" are acceptable.

    upload              Upload the package tarball to the server. This command always invokes
                        the build command automatically right before uploading.

For bug reporting instructions, please see:

    <https://openresty.org/en/community.html>

Copyright (C) Yichun Zhang (agentzh). All rights reserved.
```

* **搜索安装包**

```shell
# 搜索lua-resty-iputils包
opm search lua-resty-iputils
# 安装lua-resty-iputils包
opm get hamishforbes/lua-resty-iputils
```

## 二、使用luarocks安装

* macOS安装luarocks

```shell
brew install luarocks
```

* 搜索安装包

```shell
# 搜索包
luarocks search lua-resty-iputils
# 安装包
luarocks install lua-resty-iputils
```

**参考**

[Lua 的模块安装和部署工具 - LuaRocks - 简书](https://www.jianshu.com/p/2200b5036de1)

[基于openresty的ip白名单控制 - 荣锋亮 - 博客园](https://www.cnblogs.com/rongfengliang/p/13842909.html)
