# Sublime Text C/C++ `clangd` LSP Tips

The following takes `Ubuntu Cosmic (18.04)` as an example.

Note that the [cquery](https://github.com/cquery-project/cquery) LSP does quite the same thing. 
The key is that you have to generate a `compile_commands.json` for your project.


## Installation Prerequisites

- Install `clangd` from the apt [repositories](https://apt.llvm.org/)

  1. Add the GPG key: `$ wget -O - https://apt.llvm.org/llvm-snapshot.gpg.key | sudo apt-key add -`
  1. Add the LLVM apt repository as listed on the above web page:
  
     You have to change the following command depending on your Ubuntu version.
  
     ```bash
     sudo su -c "cat > /etc/apt/sources.list.d/llvm-toolchain-cosmic.list <<EOF
     deb http://apt.llvm.org/cosmic/ llvm-toolchain-cosmic-8 main
     deb-src http://apt.llvm.org/cosmic/ llvm-toolchain-cosmic-8 main
     EOF"
     ```
  
  1. Fetch information from the newly added repository: `$ sudo apt update`
  1. Install clang-tools: `$ sudo apt install clang-tools-8`
  1. Prefer using the installed clangd: `$ sudo update-alternatives --install /usr/bin/clangd clangd /usr/bin/clangd-8 100`
  1. Make sure `clangd` is available with the correct version: `$ clangd --version`
  
     ```
     clangd version 8.0.0-svn356034-1~exp1~20190313093039.54 (branches/release_80)
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

1. Install `LSP` via Package Control.
1. Here's a `clangd` LSP settings example:

   ```javascript
   {
     "clients": {
       "clangd": {
         "enabled": true,
         "command": [
           "clangd", // you may use an absolute path
           "-header-insertion-decorators=0",
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
