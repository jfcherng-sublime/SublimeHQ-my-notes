# Sublime Text C/C++ `clangd` LSP tips


## Installation Prerequisites

- Install `clangd` from the apt [repositories](https://apt.llvm.org/)

  - Add the GPG key: `# wget -O - https://apt.llvm.org/llvm-snapshot.gpg.key|sudo apt-key add -`
  - Install the stable branch: `# apt install clang-tools-7`
  - Now `clangd-7` should be available: `$ which clangd-7`


## Generate the Setup for Your Project

You would have to generate a `compile_commands.json` file.

- If your project uses `cmake`,
  `$ cmake path/to/source -DCMAKE_EXPORT_COMPILE_COMMANDS=ON` 
  should generate a `compile_commands.json` in the current directory.
  
- If your project uses `make`,
  use [compiledb](https://github.com/nickdiego/compiledb) to generate it. 
  You may check the readme on `compiledb`'s GitHub repository.
  
  - Install `compiledb`: `# pip install compiledb`
  - `compiledb` is just a wrapper of `make`. Use `compiledb make` as if you are using `make`. 
    So you may probably do `compiledb make all` in your `Makefile` directory.
  - `compile_commands.json` should be generated at the same directory in the above step.

The generated `compile_commands.json` usually has no information about header files, 
so you would see nonsense output from the LSP. 
But we could add header files information to `compile_commands.json`.

- Install [compdb](https://github.com/Sarcasm/compdb): `# pip install compdb`
- Assuming a build directory `build/`, containing a `compile_commands.json`, 
  a new compilation database, containing the header files, 
  can be generated with: `$ compdb -p build/ list > compile_commands.json`. 
  You may check the readme on `compdb`'s GitHub repository.

After we have a decent `compile_commands.json`, it's all set now.
Copy the generated `compile_commands.json` to your project root.
You may want to add `compile_commands.json` into you `.gitignore` as well.


## Setup for Sublime Text LSP

- Install `LSP` via Package Control.
- Here's a `clangd` LSP settings example:

```javascript
{
    "clients": {
        "clangd": {
            "enabled": true,
            "command": [
                "clangd", // may be "clangd-7" or you could soft-link it to "clangd"
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


## References

- https://lsp.readthedocs.io/en/latest/#cc-clangd
- https://github.com/tomv564/LSP/issues/398
- https://github.com/nickdiego/compiledb
- https://github.com/Sarcasm/compdb#generating-a-compilation-database-including-header-files
