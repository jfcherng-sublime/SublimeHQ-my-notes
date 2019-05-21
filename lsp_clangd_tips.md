# Sublime Text C/C++ `clangd` LSP Tips

The following takes `Ubuntu Bionic (18.04)` as an example.


## Installation Prerequisites

- Install `clangd` from the apt [repositories](https://apt.llvm.org/)

  1. Add the GPG key: `$ wget -O - https://apt.llvm.org/llvm-snapshot.gpg.key | sudo apt-key add -`
  1. Add the LLVM apt repository as listed on the above web page:
  
     You have to change the following command depending on your Ubuntu version.
     Please check the apt repositories' web page as described above.
     The minimal LLVM version required is `7`. Here I use version `8` anyway.
  
     ```bash
     sudo su -c "cat > /etc/apt/sources.list.d/llvm-toolchain-bionic.list <<EOF
     deb http://apt.llvm.org/bionic/ llvm-toolchain-bionic-8 main
     deb-src http://apt.llvm.org/bionic/ llvm-toolchain-bionic-8 main
     EOF"
     ```
  
  1. Fetch information from the newly added repository: `$ sudo apt update`
  1. Install clang-tools: `$ sudo apt install clang-tools-8`
  1. Prefer using the installed clangd: `$ sudo update-alternatives --install /usr/bin/clangd clangd /usr/bin/clangd-8 100`
  1. Make sure `clangd` is available with the correct version: `$ clangd --version`
  
     ```
     clangd version 8.0.1-svn360950-1~exp1~20190517004233.70 (branches/release_80)
     ```


## Generate the Setup for Your Project

You would have to generate a `compile_commands.json` file.

- If your project uses `cmake`,
  `$ cmake path/to/source -DCMAKE_EXPORT_COMPILE_COMMANDS=ON` 
  should generate a `compile_commands.json` in the current directory.
  
- If your project uses `make`,
  use [compiledb](https://github.com/nickdiego/compiledb) to generate it. 
  You may check the readme on `compiledb`'s GitHub repository.
  
  1. Install `compiledb`: `$ sudo pip install compiledb`
  1. `compiledb` is just a wrapper of `make`. Use `compiledb make` as if you are using `make`. 
     So you may probably do `compiledb make all` in your `Makefile` directory.
  1. `compile_commands.json` should be generated at the same directory in the above step.

The generated `compile_commands.json` usually has no information about header files, 
so you would see nonsense output from the LSP. 
But we could add header files information to `compile_commands.json`.

1. Install [compdb](https://github.com/Sarcasm/compdb): `$ sudo pip install compdb`
1. Assuming a build directory `build/`, containing a `compile_commands.json`, 
   a new compilation database, containing the header files, 
   can be generated with: `$ compdb -p build/ list > compile_commands.json`. 
   You may check the readme on `compdb`'s GitHub repository.

After we have a decent `compile_commands.json`, it's all set now.
Copy the generated `compile_commands.json` to your project root.
You may want to add `compile_commands.json` into you `.gitignore` as well.


## Setup for Sublime Text LSP

1. Install [LSP](https://packagecontrol.io/packages/LSP) via Package Control.
1. Here's a `clangd` LSP settings example:

   ```javascript
   {
     "clients": {
       "clangd": {
         "enabled": true,
         "command": [
           "clangd", // you may use an absolute path for this clangd executable
           "-function-arg-placeholders=0",
           "-header-insertion-decorators=1",
           "-index",
         ],
         "scopes": ["source.c", "source.c++", "source.objc", "source.objc++"],
         "syntaxes": ["Packages/C++/C.sublime-syntax", "Packages/C++/C++.sublime-syntax", "Packages/Objective-C/Objective-C.sublime-syntax", "Packages/Objective-C/Objective-C++.sublime-syntax"],
         "languageId": "cpp",
       },
     },
   }
   ```

1. If you have other related linters enabled, you may want to disable them since LSP is more powerful.
   To do that in your project, edit the project settings (Menu > Project > Edit Project):

   ```javascript
   {
     "folders":
     [
       // ... not important, here are just your project folders
     ],
     "settings":
     {
       // for example, to disable some other C/C++ linters
       "SublimeLinter.linters.gcc.disable": true,
       "SublimeLinter.linters.clang.disable": true,
       "SublimeLinter.linters.clang++.disable": true,
     }
   }
   ```


## References

- https://lsp.readthedocs.io/en/latest/#cc-clangd
- https://github.com/tomv564/LSP/issues/398
- https://github.com/nickdiego/compiledb
- https://github.com/Sarcasm/compdb#generating-a-compilation-database-including-header-files
- https://clang.llvm.org/extra/clangd/Installation.html


# `clangd` on Windows?

I hardly write C/C++ codes on Windows but I did make some trying.
The official LLVM/Clang support on Windows is for MSVC-only.
So I use partial MSVC + LLVM/Clang combination to make `clangd` work.

1. Donwload the [Visual Studio Build Tools 2017](https://visualstudio.microsoft.com/thank-you-downloading-visual-studio/?sku=BuildTools&rel=15) (maybe 2019 works as well) `vs_buildtools.exe`.
1. Execute `vs_buildtools.exe` and install `VC++ build tools`, `Windows 10 SDK` and `CMake VC++ tools` 
   as shown in this [screenshot](https://raw.githubusercontent.com/jfcherng/my-Sublime-Text-notes/master/images/windows-vs_buildtools-for-clangd.png).
1. Download the pre-built LLVM binaries from [here](http://releases.llvm.org/download.html) or even a nightly build from [here](https://llvm.org/builds/).
   Remember that the minimal version has to be `7` for `clangd` to work properly. I use a nightly build here anyway.
1. Install the downloaded LLVM installer.
1. Reboot your PC.
1. I do not do any extra setup and `clangd` is working now.
   I use the same LSP settings with the one I used in `Ubuntu` above.
   Of course, you still have to deal with generating a `compile_commands.json`
   which seems to be a harder part on Windows.


# How about the `cquery` LSP?

[cquery](https://github.com/cquery-project/cquery) LSP does quite the same thing. 
The key is that you have to generate a `compile_commands.json` for your project. 
But, [it looks like](https://github.com/cquery-project/cquery/issues/867) `cquery` 
has been abandoned due to its inactivity and the thriving of `clangd`.
