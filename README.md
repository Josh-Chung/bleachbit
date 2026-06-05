# BleachBit

BleachBit cleans files to free disk space and to maintain privacy.

## Running from source

To run BleachBit without installation, unpack the tarball and then run these
commands:

    make -C po local # build translations
    python3 bleachbit.py

Then, review the preferences.

Then, select some options, and click Preview.  Review the files, toggle options accordingly, and click Delete.

For information regarding the command line interface, run:

     python3 bleachbit.py --help

Read more about [running from source](https://docs.bleachbit.org/dev/running-from-source-code.html).

### Windows 用户：从源码启动 GUI

如果你在 Windows 上希望**直接从源码运行 BleachBit GUI**（无需构建 .exe），请参阅 **[TUTORIAL.md](doc/TUTORIAL.md)**。该教程详细介绍了如何通过 MSYS2 MINGW64 环境配置所有依赖（GTK3、PyGObject 等），并提供一键启动脚本，让你可以像使用安装版一样直接运行源码。

## Links

* [BleachBit home page](https://www.bleachbit.org)
* [Support](https://www.bleachbit.org/help)
* [Documentation](https://docs.bleachbit.org)

## Localization

Read [translation documentation](https://www.bleachbit.org/contribute/translate) or translate now in [Weblate](https://hosted.weblate.org/projects/bleachbit/), a web-based translation platform.

<a href="https://hosted.weblate.org/engage/bleachbit/">
      <img src="https://hosted.weblate.org/widgets/bleachbit/-/multi-auto.svg" alt="Translation status"/>
</a>

## Licenses

BleachBit itself, including source code and cleaner definitions, is licensed under the [GNU General Public License version 3](COPYING), or at your option, any later version.

markovify is licensed under the [MIT License](https://github.com/jsvine/markovify/blob/master/LICENSE.txt).

## Development
* [BleachBit on AppVeyor](https://ci.appveyor.com/project/az0/bleachbit)  ![Build status](https://ci.appveyor.com/api/projects/status/7p8amofd7rv7n268?svg=true)
* [CleanerML Repository](https://github.com/bleachbit/cleanerml)
* [BleachBit Miscellaneous Repository](https://github.com/bleachbit/bleachbit-misc)
* [Winapp2.ini Repository](https://github.com/bleachbit/winapp2.ini)

### Docker-based Linux builds

The `docker/build.sh` script sets up a container, runs `make tests`,
and builds packages for a selected distribution. Usage:

```
./docker/build.sh <distro> # distro is one of {debian, fedora, opensuse}
```

Artifacts are copied to `./docker-artifacts/<distro>` on the host.