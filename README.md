# wrapin-py  

Pronounced: _Wrap-In-Pie_  

Wraps an executable binary file inside a Python source file and delegates calls to the wrapped binary, enabling the usage of a compiled executable in a sandboxed or jailed execution environment that is usually limited to a python script.

`wrapin-py` is cross-platform and supporting both Python 2 and Python 3 environments.

## Liability

### __*Try at your own risk!*__  

Although this "hack" does not attempt to do anything that requires high-privilege, neither does it use any non-conventional privilege-escalation technics, I cannot guarantee that it will work on your system or that it will not break anything.

---

## But, Why?

<details>
<summary>Project Motivations (click to expand)</summary>

## Project Motivations

This project serves me as a research and a Proof-of-Concept, about the idea of using Rust as a scripting language for platforms that usually support scripting with interpreted languages. Hopefully, scripting directly in the Rust programming language, will be a thing of the future for closed systems as well.

Languages used for scripting within closed source products are usually running inside a very limited sandbox or jailed environment, which at best, is able to offer us usage of the base libraries of the language or maybe even a few selected third-parties (e.g. `requests`). Also, some environments are still using Python 2 and do not support more advanced features, such as Python annotations that can help reduce errors (when used correctly) by guiding our IDE to catch misuse mistakes early on.

For the most part, Python is still powerful, easy, and good enough for about any scripting task. But for some more advanced use case scenarios, when logic can get real complex with advanced features, an alternative may fit better.

As I worked with the Rust programming language recently, I realized it has some big advantages that could serve me well when my projects (scripts or otherwise) get long and complex with their logical flows. An advantage well noticed when returning to an old Python project (even when it has a good or even a great design). This ultimately lead me to wonder if using Rust, a statically-compiled, strongly-typed and a new generation of a low-level language (comparable to C\C++ on steroids), could serve me for a complex script design while using its powerful compiler as an excellent guarantee for my code correctness. Modifying and maintaining a complex Rust program gets me a peace of mind as I know that if it compiles, it is correct.

</details>

<details>
<summary>Advantages of a Self-Contained low-level Binary (click to expand)</summary>

## Advantages of a Self-Contained low-level Binary

1. __*Single file*__, independent of any runtime environment, can be embedded with third-party libraries while offering more advanced APIs, such that are not part of the core Python libraries which are provided by default in the sandboxed environment  
   - __Caveat:__ In restricted, stripped environments, this can work as long as your binary does not try to use dynamically linked APIs which may be restricted or completely missing in that sandboxed environment (e.g. `cmake` or native `OpenSSL`).

2. Written in a strongly typed, statically compiled programming language for a complex project, can detect potential problems at compile-time, long before they reach production, especially for the rare, "invisible" bug cases that camouflage themselves within tons of code so they can show up at the worse moment in production.

3. It can provide you with bare-metal performance for some heavy calculations.

</details>

---

## Usage

How to use the `wrapin.py` tool.

<details>
<summary>Help (click to expand)</summary>

### Help

You can find the most updated command list in the help menu, using the `--help` switch.

```bash
python wrapin.py --help
```

#### Output

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

<details>
<summary>Wrap an Executable (click to expand)</summary>

### Wrap an Executable

```powershell
python wrapin.py hello_world_windows.exe
```

Will produce the wrapper file: `hello_world_windows.exe.wrapped.py`

📦 (Working directory)  
 ┣ 📜hello_world_windows.exe  
 ┣ 📜hello_world_windows.exe.wrapped.py  👈  
 ┗ 📜wrapin.py

- Upload the wrapper file as an automations script to your external system.

</details>  

<details>
<summary>Wrap an Executable for a different Architecture (click to expand)</summary>

### Wrap an Executable for a different Architecture

The next is an example of wrapping a *Linux* binary file from a *Windows* environment

```powershell
python wrapin.py hello_world_linux.bin --target=linux
```

- Providing a wrong platform, e.g. running the wrapper file in a Windows environment while it is configured for Linux, will simply exit with an error, without trying to run the wrapped executable.  

</details>

<details>
<summary>Logical Flow Graph (click to expand)</summary>

## Logical Flow Graph

### Wrap & Upload the Executable

![Alt text](./assets/wrapping-diagram.svg)

### Auto-Unwrap & Delegate Script Calls

![Alt text](./assets/unwrapping-diagram.svg)

- The wrapped file is unwrapped into a directory called `unwrapped` that is automatically created within the logged machine user's home directory (`~`). This is done to make sure that a lack of permissions will not pose a problem.

</details>

---

## Language Guides

Tips and tricks for specific programming languages to create a proper binary file to wrap.

Select the language of choice:

<details>
<summary>Rust (click to expand)</summary>

### Optimize for Size

Unless you have some very specific requirements, it is recommended for you to optimize your binary for size in the context of scripting.

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

---

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

---

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

---

## Test List

*This list only indicates what has been tested and proven to work so far and does not reflect possible usability for other systems*

- QRadar Community Edition 7.3.3 (`Custom Actions`)
  - Rust binary file, compiled for  `i686-unknown-linux-musl`

## Contribution  

Tried it on a system that is not listed here and it worked?  
Feel free to add that system to the list in a pull request!  
*Just don't forget to mention how you achieved that, preferably in a markdown language that can later be converted into a guide.*

Feel free to contribute in anyway you like:

- Discussions  
- Issues (Report bugs, mistakes, feature requests, etc.)  
- Pull Requests  
- Blog Posts  
- Guides  
- etc.  

## License

This project and any direct contribution to it are licensed under the MIT license.

Refer to the `LICENSE` file in this repository for more details.
