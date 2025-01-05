# Shell

## yunmingOS 概述

yunmingOS 是一个基于 ArceOS 操作系统的轻量级嵌入式操作系统，旨在提供一个简单、有效的 Shell 环境。它支持基本的文件系统操作、命令解析、以及通过 Shell 交互来执行各种命令。系统的核心是基于 Rust 编程语言实现，借助了 ArceOS 的底层支持，并通过 axstd 提供了对标准库的部分替代。

该操作系统设计为内存中的文件系统 (RamFS)，通过内存分配实现虚拟磁盘的文件管理，并提供类似 Linux 的命令行工具，供开发者和用户进行交互。

## 依赖与组件

### ArceOS

ArceOS 是一个轻量级的操作系统内核，它为 yunmingOS 提供了基础的系统服务，如任务调度、内存管理、硬件抽象等。ArceOS 以其简洁的设计和高效的执行性能为特点，适用于资源受限的嵌入式设备。yunmingOS 利用了 ArceOS 提供的接口来实现文件系统操作、进程管理、以及其他系统级功能。

### AxStd（标准库）

为了替代标准的 Rust 库，yunmingOS 使用了 axstd，这是一个为嵌入式环境定制的标准库。axstd 提供了对 I/O、字符串处理、环境变量、文件系统等常用功能的支持，但不依赖于底层的操作系统 API，因此具有较高的移植性和可控性。通过 axstd，开发者能够更轻松地进行系统级编程，同时保持较低的系统开销。

### RamFS（内存文件系统）

yunmingOS 使用了内存文件系统（RamFS）来管理文件操作。RamFS 将文件系统的元数据和文件内容存储在内存中，极大地提高了文件操作的速度，但也限制了文件系统的持久性。通过 RamFS，系统能够在不依赖外部存储设备的情况下进行文件读写操作。文件系统的实现是基于 axfs_ramfs 和 axfs_vfs 提供的文件系统接口，实现了基本的文件创建、删除、读取等功能。

在此架构中，文件的读取和写入会直接操作内存中的数据块，而非物理硬盘，适合于对速度要求高且不需要持久存储的场景。

## 系统模块

yunmingOS 由多个模块组成，每个模块负责不同的任务。以下是系统的核心模块：

1. **Shell**：提供命令行界面，允许用户通过输入命令与操作系统交互。Shell 负责命令的解析、执行和输出显示。
1. **内存文件系统（RamFS）**：处理文件和目录的管理，提供文件操作接口。
1. **命令解析器**：负责解析用户输入的命令，匹配相应的命令处理函数，并执行操作。
1. **命令实现**：包括基本的文件操作命令（如 `ls`、`cat`、`mkdir` 等），以及系统管理命令（如 `help`、`exit`）。
1. **I/O 管理**：处理输入输出流，支持从标准输入读取用户命令，向标准输出显示结果。

通过这些模块的协作，yunmingOS 能够提供一个轻量、灵活的操作环境，满足基本的文件管理需求。

## 系统设计原则

yunmingOS 设计时秉持以下原则：

- **简洁性**：系统尽量保持简单，减少不必要的复杂性，专注于实现基本功能。
- **高效性**：针对资源受限的设备，进行性能优化，减少内存使用和计算开销。
- **可移植性**：通过 axstd 和 ArceOS 提供的抽象，确保系统能够在不同硬件平台上运行。
- **模块化**：系统模块之间的低耦合使得功能的扩展和维护变得更加简便。

通过这些原则，yunmingOS 在实现嵌入式设备的基本功能时，能够提供一个高效、易于维护的操作环境。

好的，以下是第二部分关于 **Shell 设计** 的详细内容：

## Shell 设计

### Shell 功能概述

在 yunmingOS 中，Shell 提供了一个交互式命令行接口，允许用户与操作系统进行交互。用户可以输入各种命令，Shell 会解析并执行相应的操作。Shell 的设计目标是简洁、易用，并能高效地处理用户的命令请求。

Shell 实现了类似 Unix/Linux 系统中的基本命令，如 `ls`、`cat`、`cd`、`echo` 等，同时支持文件系统操作和简单的控制流功能。Shell 的主要功能包括：

- 显示命令提示符：提示用户输入命令。
- 读取用户输入：通过标准输入获取命令字符串。
- 解析命令：将用户输入的命令拆分为命令名和参数，并查找对应的命令处理函数。
- 执行命令：根据命令解析结果调用相应的命令处理函数。
- 输出结果：将执行结果输出到屏幕或其他设备。

Shell 作为用户与操作系统之间的桥梁，提供了一种方便的交互方式，用户可以通过它快速完成各种操作。

### 命令解析与处理流程

命令解析是 Shell 的核心功能之一，它负责将用户输入的命令字符串转换为可以执行的操作。以下是命令解析和处理的流程：

1. **显示提示符并读取输入**：Shell 会先显示命令提示符（如 `yunmingos:~/current_directory$ `），等待用户输入命令。

   ```rust
   print_prompt();
   ```

   然后，Shell 从标准输入读取字符并将其存储在缓冲区中，直到用户按下回车键。

1. **解析命令**：输入的命令字符串会被拆分成命令名和参数。在解析时，Shell 会忽略空格、换行符等无关字符，将命令名提取出来，然后将剩余的部分作为参数传递给命令处理函数。

   ```rust
   let line_str = unsafe { core::str::from_utf8_unchecked(line) };
   let (cmd, args) = split_whitespace(line_str);
   ```

   `split_whitespace` 函数会根据空白字符（如空格、制表符）将命令行分割成命令和参数。

1. **匹配命令**：Shell 会遍历内置的命令表（`CMD_TABLE`），匹配输入的命令名，找到对应的命令处理函数。

   ```rust
   for (name, func) in CMD_TABLE {
       if cmd == *name {
           func(args);
           return;
       }
   }
   ```

   如果找到了匹配的命令，Shell 会调用该命令的处理函数，并将参数传递给它。如果没有找到匹配的命令，Shell 会输出错误信息并提示用户可用的命令列表。

1. **执行命令**：命令执行的具体操作由相应的命令处理函数实现。例如，`ls` 命令会列出当前目录的文件，`cat` 命令会显示文件内容，`cd` 命令会更改当前目录。

   ```rust
   fn do_ls(args: &str) {
       // 列出文件和目录
   }

   fn do_cat(args: &str) {
       // 显示文件内容
   }
   ```

   每个命令处理函数内部会根据传入的参数执行不同的文件操作或系统调用。

1. **输出结果**：命令执行后，Shell 会将结果输出到标准输出（通常是终端）。例如，`ls` 命令会显示文件列表，`echo` 命令会显示文本。

   ```rust
   println!("{}", result);
   ```

### 输入输出处理

Shell 的输入输出处理是交互式命令行的关键部分。用户输入的每一个字符都会被实时处理并反馈给用户，输出结果也会立即显示在终端上。具体实现如下：

1. **输入处理**：

   - 输入的字符会被存储在缓冲区中，Shell 会实时读取字符并根据用户的操作（如删除、换行、回车）做相应处理。
   - 当用户按下回车键时，输入的命令会被提交给 Shell 进行解析和处理。如果是有效的命令，Shell 会执行相应的操作；否则，会输出错误提示。
   - Shell 还支持字符删除（使用退格键）和命令补全等基本的编辑功能。

   ```rust
   if stdin.read(&mut buf[cursor..cursor + 1]).ok() != Some(1) {
       continue;
   }

   match buf[cursor] {
       CR | LF => {
           // 处理换行或回车
       }
       BS | DL => {
           // 处理退格
       }
       0..=31 => {}
       c => {
           // 处理输入字符
           stdout.write_all(&[c]).unwrap();
       }
   }
   ```

1. **输出处理**：

   - Shell 会将命令执行的结果输出到标准输出。如果执行的是文件操作命令（如 `ls`、`cat`），结果会直接显示在屏幕上。
   - 输出过程中，Shell 会确保格式化正确，并根据需要进行换行、清屏等操作。

   ```rust
   println!("{}", output);
   ```

   对于一些命令（如 `echo`），如果有文件重定向（如 `echo "text" > file.txt`），Shell 会将输出内容写入文件，而不是显示在终端上。

   ```rust
   let mut file = File::create(fname)?;
   file.write_all(text.as_bytes())?;
   ```

### 命令执行与响应

命令的执行是通过调用相应的命令处理函数来实现的。每个命令处理函数都有其特定的功能，并接受命令参数进行操作。执行结果通过标准输出或标准错误输出返回给用户。

1. **命令处理函数**：每个命令（如 `ls`、`cd`、`cat` 等）都有一个对应的处理函数。处理函数会根据参数的不同执行不同的操作。例如，`ls` 命令列出目录内容，`cat` 命令显示文件内容，`cd` 命令切换当前目录。

    ```rust
    fn do_ls(args: &str) {
        // 查看文件树
    }

    fn do_cat(args: &str) {
        // 显示文件内容
    }
    ```

1. **错误处理**：如果命令执行过程中出现错误（如文件未找到、权限不足等），Shell 会捕获错误并输出错误信息。

   ```rust
   println!("{}: command not found", cmd);
   ```

1. **多行命令支持**：Shell 还支持多行命令的输入。例如，当用户输入带有换行符的命令时，Shell 会提示用户继续输入直到命令完整。

   ```rust
   if cursor > 0 && buf[cursor - 1] == BACKSLASH {
       cursor -= 1;
       stdout.write_all(&[b'>']).unwrap();
       multiline = true;
       continue;
   }
   ```

1. **命令结束与提示符**：命令执行完毕后，Shell 会重新显示命令提示符，等待用户输入下一个命令。

   ```rust
   print_prompt();
   ```

通过这些处理流程，Shell 能够有效地管理用户的输入与输出，确保命令执行的顺利进行并向用户提供及时的反馈。

## 模块分析

### main.rs

#### 启动流程

`main.rs` 是程序的入口点，它负责启动 Shell 的基本流程，包括初始化内存、设置文件系统、以及启动命令循环等。启动流程的主要步骤如下：

1. **初始化系统**：
   - 首先，`main.rs` 会初始化一些必要的系统资源，如内存管理、硬件初始化等。
   - 设定文件系统的根目录，加载基本的文件系统结构。

   ```rust
   fn main() {
       init_memory();
       init_filesystem();
       run_shell();
   }
   ```

2. **运行 Shell**：
   - 初始化完成后，`main.rs` 会调用 `run_shell` 函数，进入 Shell 命令循环。
   - 这个函数会不断循环，等待用户输入命令并解析执行。

   ```rust
   fn run_shell() {
       loop {
           print_prompt();        // 打印命令提示符
           let input = read_input();  // 读取用户输入
           process_command(input);   // 处理用户命令
       }
   }
   ```

#### 输入循环与命令处理

在 `main.rs` 中，Shell 会使用一个输入循环来持续读取用户的输入，并对输入的命令进行处理。输入循环的基本步骤如下：

1. **读取输入**：通过 `read_input` 函数从标准输入流获取用户输入的命令行。

   ```rust
   fn read_input() -> String {
       let mut input = String::new();
       std::io::stdin().read_line(&mut input).expect("Failed to read line");
       input.trim().to_string()  // 去掉多余的换行符
   }
   ```

2. **命令解析与执行**：获取输入后，Shell 会通过 `process_command` 函数解析命令，并调用相应的命令处理函数。

   ```rust
   fn process_command(input: String) {
       let parts: Vec<&str> = input.split_whitespace().collect();
       match parts[0] {
           "ls" => ls(parts),
           "cat" => cat(parts),
           "cd" => cd(parts),
           _ => println!("Unknown command: {}", parts[0]),
       }
   }
   ```

#### 提示符打印与多行命令支持

Shell 会定期打印命令提示符，等待用户输入。提示符的打印是通过 `print_prompt` 函数实现的。支持多行命令输入的功能可以通过判断用户输入的换行符来触发。

1. **打印提示符**：每次循环开始时，Shell 会打印当前路径作为命令提示符。

   ```rust
   fn print_prompt() {
       let current_path = get_current_path(); // 获取当前路径
       println!("{}$ ", current_path);
   }
   ```

2. **多行命令支持**：Shell 允许用户在一行输入多条命令，或输入不完整的命令，用户需要继续输入直到命令完整。通过判断命令中的反斜杠 `\` 符号来支持多行命令输入。

   ```rust
   fn handle_multiline_input(input: &mut String) {
       if input.ends_with("\\") {
           print!("> ");  // 打印换行提示符
           input.push_str(&read_input());
       }
   }
   ```

### ramfs.rs

#### 内存文件系统的实现

`ramfs.rs` 实现了一个内存文件系统（RAMFS），提供了一个在内存中创建和管理文件的简易机制。这使得 Shell 可以快速进行文件操作而不需要依赖磁盘存储。内存文件系统的实现通常包括：

1. **文件节点结构**：
   每个文件或目录都有一个文件节点结构，存储文件的元数据（如文件名、大小、类型等）。

   ```rust
   pub struct FileNode {
       pub name: String,
       pub content: Vec<u8>, // 文件内容存储在内存中
       pub is_dir: bool,
   }
   ```

2. **文件系统接口定义与实现**：
   `ramfs.rs` 提供了文件系统接口，定义了文件创建、删除、读取、写入等操作，并将这些操作映射到内存文件系统的实现上。

   ```rust
   pub fn create_file(name: &str, is_dir: bool) -> FileNode {
       FileNode {
           name: name.to_string(),
           content: Vec::new(),
           is_dir,
       }
   }

   pub fn read_file(file: &FileNode) -> Option<Vec<u8>> {
       if file.is_dir {
           None
       } else {
           Some(file.content.clone())
       }
   }

   pub fn write_file(file: &mut FileNode, data: Vec<u8>) {
       if !file.is_dir {
           file.content = data;
       }
   }
   ```

### cmd.rs

#### 命令表与命令处理

`cmd.rs` 定义了命令表以及相应的命令处理函数。命令表是一个映射，包含了所有支持的命令名称和对应的命令处理函数。通过遍历命令表，Shell 可以根据用户输入的命令调用相应的处理函数。

```rust
pub const CMD_TABLE: &[(&str, fn(&str))] = &[
    ("ls", do_ls),
    ("cat", do_cat),
    ("echo", do_echo),
    ("mkdir", do_mkdir),
    ("pwd", do_pwd),
    ("rm", do_rm),
    ("cd", do_cd),
    ("touch", do_touch),
    ("clear", do_clear),
    ("exit", do_exit),
    ("help", do_help),
];
```

#### 各种命令实现

在 `cmd.rs` 中，Shell 的各种命令被定义为函数，并接受命令参数进行相应的操作。以下是一些常见命令的实现：

##### ls：列出目录内容

`ls` 命令列出当前目录下的文件和子目录。

```rust
fn do_ls(args: &str) {
    let current_dir = get_current_dir();
    let files = list_files(current_dir);
    for file in files {
        println!("{}", file.name);
    }
}
```

##### cat：查看文件内容

`cat` 命令用于显示文件的内容。

```rust
fn do_cat(args: &str) {
    let file = open_file(args);
    match file {
        Some(f) => {
            for byte in f.content.iter() {
                print!("{}", *byte as char);
            }
        }
        None => println!("File not found: {}", args),
    }
}
```

##### echo：输出或重定向内容

`echo` 命令输出文本，支持重定向到文件。

```rust
fn do_echo(args: &str) {
    if args.contains(">") {
        let parts: Vec<&str> = args.split(">").collect();
        let content = parts[0].trim();
        let filename = parts[1].trim();
        write_to_file(filename, content);
    } else {
        println!("{}", args);
    }
}
```

##### mkdir：创建目录

`mkdir` 命令创建一个新目录。

```rust
fn do_mkdir(args: &str) {
    let dir_name = args.trim();
    create_directory(dir_name);
}
```

##### pwd：显示当前路径

`pwd` 命令显示当前工作目录的路径。

```rust
fn do_pwd(args: &str) {
    let current_path = get_current_path();
    println!("{}", current_path);
}
```

##### rm：删除文件/目录

`rm` 命令删除指定的文件或目录。

```rust
fn do_rm(args: &str) {
    let target = args.trim();
    remove_file_or_dir(target);
}
```

##### cd：切换当前目录

`cd` 命令切换当前工作目录。

```rust
fn do_cd(args: &str) {
    let dir_name = args.trim();
    change_directory(dir_name);
}
```

##### touch：创建空文件或更新文件时间戳

`touch` 命令创建一个空文件，或者更新现有文件的时间戳。

```rust
fn do_touch(args: &str) {
    let file_name = args.trim();
    touch_file(file_name);
}
```

##### clear：清屏

`clear` 命令清除终端屏幕。

```rust
fn do_clear(args: &str) {
    println!("\x1B[2J\x1B[H");  // ANSI escape code 清屏
}
```

##### exit：退出 Shell

`exit` 命令退出 Shell 会话。

```rust
fn do_exit(args: &str) {
    println!("Exiting shell...");
    std::process::exit(0);
}
```

##### help：显示帮助信息

`help` 命令显示所有支持的命令和简要说明。

```rust
fn do_help(args: &str) {
    println!("Available commands:");
    println!("ls - List directory contents");
    println!("cat - Concatenate and display file contents");
    println!("echo - Output text to the terminal");
    println!("mkdir - Create a new directory");
    println!("pwd - Show current working directory");
    println!("rm - Remove a file or directory");
    println!("cd - Change the current directory");
    println!("touch - Create an empty file or update the timestamp");
    println!("clear - Clear the terminal screen");
    println!("exit - Exit the shell");
    println!("help - Show this help message");
}
```


## 扩展与自定义

### 支持的新命令与功能扩展

Shell 可以通过添加新的命令来扩展功能。常见的扩展命令包括：

- **find**：在当前目录及子目录中查找文件。
- **重定向 `>>`**：支持将输出追加到文件中。
- **用户管理**：如 `useradd` 和 `userdel`，模拟简单的用户管理功能。

示例：扩展 `echo` 支持追加到文件

```rust
fn do_echo(args: &str) {
    if args.contains(">") {
        let parts: Vec<&str> = args.split(">").collect();
        let content = parts[0].trim();
        let filename = parts[1].trim();
        append_to_file(filename, content);
    } else {
        println!("{}", args);
    }
}

fn append_to_file(filename: &str, content: &str) {
    let mut file = open_or_create_file(filename);
    file.content.extend_from_slice(content.as_bytes());
}
```

### 定制文件系统与接口的实现

Shell 可以定制文件系统实现，包括：

- **内存文件系统（RAMFS）扩展**：支持软链接和文件权限。
- **网络文件系统**：模拟通过网络操作文件。

例如，支持软链接：

```rust
pub struct FileNode {
    pub name: String,
    pub is_symlink: bool,
    pub symlink_target: Option<String>,
}

fn create_symlink(target: &str, link_name: &str) -> FileNode {
    FileNode {
        name: link_name.to_string(),
        is_symlink: true,
        symlink_target: Some(target.to_string()),
    }
}
```

也可以实现 **数据库文件系统**：

```rust
pub trait FileSystem {
    fn create_file(&self, path: &str) -> bool;
    fn read_file(&self, path: &str) -> Option<String>;
}

pub struct DatabaseFileSystem {
    db_connection: String,
}

impl FileSystem for DatabaseFileSystem {
    fn create_file(&self, path: &str) -> bool { true }
    fn read_file(&self, path: &str) -> Option<String> { Some("File content".to_string()) }
}
```

这样可以灵活定制文件系统与接口，实现更多的功能需求。
