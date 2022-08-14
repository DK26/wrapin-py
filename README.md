# wrapin-py

Wrap a binary executable file in a Python file and delegate python calls to that binary, enabling the usage of a compiled executable in a sandboxed or jailed execution environment for a script.

## Motivation

This project serves me as a Proof-of-Concept, and hopefully, scripting directly in the Rust programming language, will be a thing of the future for closed systems.

It is most common for interrupted languages to be used as scripting languages within closed source products, running under a very limited sandbox or jailed with mostly being able to only use the base libraries of the language. One such language is one of my favorites: __*Python*__.

For most use cases, Python is just powerful and easy enough for about any task and would work just fine.

As I worked with the Rust programming language recently, I realized it has some big advantages that could serve me when my projects (scripts or otherwise) get long and complex with their logical flows. An advantage well felt when returning to an old Python project (even when it has a good or even a great design). This ultimately lead me to wonder if using Rust, a statically-compiled, strongly-typed, new generation of a low-level language (like C\C++), could serve me for complex script designs with its powerful compiler as a guardian over my code correctness.

## Advantages of a low-level language binary

1. __*Single file*__, independent of any runtime environment, that can embed third-party libraries in a single file and offer APIs that are not part of the core Python libraries that are provided by the sandboxed environment  
   - __Caveat:__ This works as long as your binary does not try to use dynamically linked APIs which may be restricted or missing in the sandboxed environment (e.g. `cmake` or native `OpenSSL`).

2. Using a strongly typed, statically compiled programming language for complex projects, that is also highly performant.

3. Can override limitations over existing APIs causes by the sandboxed environment. e.g. Avoid dependencies that depend on dynamically linked APIs which may not be available in that environment, such as failing to use the TLS protocol in an HTTP request due to its external dependency on the native OpenSSL API which may not be available in that environment

## Liability

__*Try at your own risk!*__  
Although this "hack" does not attempt to do anything that requires high-privilege, neither does it use any non-conventional privilege-escalation technics, I cannot guarantee that it will work on your system or that it will not break anything.

## Usage

```bash
python wrapin.py --help
```

<details>
<summary>Output (click to expand)</summary>

```text
usage: wrapin.py [-h] [-o OUTPUT] [-t TARGET] binary_file

Wrap a binary file into a Python file container to be used in 
the context of scripting environments that are some what limited to Python and its core libraries.

positional arguments:
  binary_file           The binary file to be wrapped inside the Python file container.

options:
  -h, --help            show this help message and exit
  -o OUTPUT, --output OUTPUT
                        Specify the output path for the Python container file.
  -t TARGET, --target TARGET
                        Specify the target operating system for the binary file: `Windows`, `Linux` or `Darwin`. By
                        default, the current operating system is selected. Mismatch of configurations with the wrapped
                        file will cause a failure in execution and will exit with an error.
```

</details>  

### Wrap an Executable

```powershell
python wrapin.py hello_world_windows.exe
```

Will produce the next file: `hello_world_windows.exe.wrapped.py`

📦CURRENT DIRECTORY  
 ┣ 📜hello_world_windows.exe  
 ┣ 📜hello_world_windows.exe.wrapped.py  
 ┗ 📜wrapin.py

### Wrap an Executable for a different Architecture

The next is an example of wrapping a *Linux* binary file from a *Windows* environment

```powershell
python wrapin.py hello_world_linux.bin --target=linux
```

## Methodology

### Wrap & Upload the Executable

```mermaid
graph LR;
    EXEC[Executable File];
    EXEC --> |Wrap| WRAPPED;
    WRAPPED[Wrapped in '.py' file];
    WRAPPED -->|Upload| SYSTEM;
    SYSTEM[External System];
```

### Auto-Unwrap & Delegate Script Calls

```mermaid
graph LR;
    SYSTEM[External System];
    SYSTEM --> |Calls| WRAPPED;
    WRAPPED[Wrapping '.py' Script];
    WRAPPED --> TEST;
    TEST{Is Executable Unwrapped?};
    TEST ==> |Yes| RUN_EXEC;
    TEST --> |No| UNWRAP;
    RUN_EXEC(Run Executable & Pass Arguments);
    UNWRAP(Unwrap Executable);
    UNWRAP ==> |Then| RUN_EXEC;
```

The wrapped file is unwrapped into the running user's home directory under a new directory called `unwrapped`. This is done to make sure that lack of permissions will not pose a problem.

## Tested On

*This list only indicates what has been tested and proven to work so far and does not reflect possible usability for other systems*

- QRadar Community Edition 7.3.3 (`Custom Actions`)
  - Rust binary file, compiled for  `i686-unknown-linux-musl`

## Language Guides

<details>
<summary>Rust (click to expand)</summary>

### Optimize for Size

Unless you have some very specific requirements, it is recommended for you to optimize your binary for size for the context of scripting.

- Configure `Cargo.toml` to optimize the `--release` build for size  
  - Use `panic = 'abort'` to exit on panic rather than unwind, unless you are catching unwind to recover from panics in your use case

```toml
[profile.release]
panic = 'abort'
codegen-units = 1
incremental = true
lto = true
opt-level = 'z' # Optimize for size
```

### Building to 32-bit Targets (Reducing Size)

Add a 32-bit target to reduce the binary size, unless you are expected to use more than 4 GB RAM or having some other limiting reason to do so.

- If you insist on a 64-bit binary, replace all `i686` values with `x86_64`

<details>
<summary>Windows 32-bit (click to expand)</summary>

Add `i686-pc-windows-msvc` to the rustup toolchain

```powershell
rustup target add i686-pc-windows-msvc
```

Build target

```powershell
cargo build --target=i686-pc-windows-msvc --release
```

</details>

<details>
<summary>Linux 32-bit (click to expand)</summary>

Since we are most probably building for a limited jail or sandboxed environment, our best chance is to compile to `musl` instead of `gnu`, since `gnu` is using APIs that are dynamically linked and may be absent within the context of the sandbox.  

### GNU & MUSL

- You can always try `gnu` first and fallback to `musl`  
- Using `musl` requires you to use pure Rust alternative libraries. e.g. replace `rust-native-tls` with `rust-tls`  
- Don't get discouraged using `musl` if your `gnu` binary cannot access dynamically linked libraries; This could also explain why some of your python code is unable to perform certain functionalities as well. With `musl` you can get a chance to overcome these problems.

### Building Proper   

Add `i686-unknown-linux-musl` to the rustup toolchain

```bash
rustup target add i686-unknown-linux-musl
```

Build target

```bash
cargo build --target=i686-unknown-linux-musl --release
```

</details>

</details>

## Contribution  

Tried it on a system that is not listed here and it worked?  
Feel free to add that system to the list in a pull request!  
*Just don't forget to mention how you achieved that, preferably in a markdown language that can later be converted into a guide.*

Feel free to contribute in anyway you like:

- Discussions  
- Issues (Report bugs, mistakes, request features, etc.)  
- Pull Requests  
- Blog Posts  
- Guides  
- etc.  

## License

This project and any direct contribution to it are licensed under the MIT license.

Refer to the `LICENSE` file in this repository for more details.
