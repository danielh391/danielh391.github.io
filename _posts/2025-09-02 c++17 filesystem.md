---
layout:     post
title:      std::filesystem
subtitle:   
date:       2025-09-02
author:     danielh
header-img: img/post-bg-rwd.jpg
catalog: true
tags:
    - c++语法
    - c++17


---
C++17 引入的 `<filesystem>` 库（通常简称为 `std::filesystem` 或 `fs`）极大地简化了文件和目录操作。以下是常用的类和函数：

## 常用命名空间和类型别名

```cpp
#include <filesystem>
namespace fs = std::filesystem; // 常用别名

// 常用类型
fs::path;        // 表示路径的类
fs::file_status; // 文件状态信息
fs::file_type;   // 文件类型枚举
fs::perms;       // 权限枚举
```

## 核心类：`fs::path`

这是最重要的类，表示文件系统路径。

### 常用构造函数和赋值
```cpp
fs::path p1("/home/user/file.txt"); // 从字符串构造
fs::path p2 = "C:\\Windows\\System32"; // Windows路径
fs::path p3 = p1; // 拷贝构造
```

### 常用成员函数
```cpp
fs::path p = "/home/user/document.txt";

// 路径分解
p.filename();    // "document.txt"
p.stem();        // "document" (无扩展名)
p.extension();   // ".txt"
p.parent_path(); // "/home/user"
p.root_path();   // "/" (Unix) 或 "C:\\" (Windows)

// 路径操作
p.append("subdir");  // 添加路径组件
p /= "file.txt";     // 同上，更常用
p.concat(".bak");    // 字符串拼接（不添加分隔符）
p += ".bak";         // 同上

// 路径检查
p.empty();      // 是否为空路径
p.is_absolute(); // 是否是绝对路径
p.is_relative(); // 是否是相对路径

// 字符串转换
p.string();     // 返回std::string
p.wstring();    // 返回std::wstring
p.u8string();   // 返回UTF-8编码的std::string
```

## 常用自由函数

### 文件系统操作
```cpp
// 文件信息
fs::exists(path);           // 路径是否存在
fs::is_directory(path);     // 是否是目录
fs::is_regular_file(path);  // 是否是普通文件
fs::file_size(path);        // 获取文件大小（字节）
fs::last_write_time(path);  // 获取最后修改时间

// 文件操作
fs::copy(src, dst);         // 复制文件/目录
fs::create_directory(path); // 创建目录
fs::create_directories(path); // 创建目录（包括父目录）
fs::remove(path);           // 删除文件/空目录
fs::remove_all(path);       // 递归删除目录及其内容
fs::rename(old_path, new_path); // 重命名/移动

// 路径操作
fs::absolute(path);         // 获取绝对路径
fs::canonical(path);        // 获取规范化的绝对路径（解析符号链接）
fs::relative(path, base);   // 获取相对于base的路径
```

### 目录遍历
```cpp
// 遍历目录内容
for (const auto& entry : fs::directory_iterator(path)) {
    std::cout << entry.path() << std::endl;
}

// 递归遍历目录
for (const auto& entry : fs::recursive_directory_iterator(path)) {
    std::cout << entry.path() << std::endl;
}

// directory_entry的常用方法
auto entry = *fs::directory_iterator(path).begin();
entry.path();       // 完整路径
entry.is_directory(); // 是否是目录
entry.file_size();  // 文件大小
```

## 常用枚举类型

### 文件类型 `fs::file_type`
```cpp
fs::file_type type = fs::status(path).type();

switch(type) {
    case fs::file_type::regular:    // 普通文件
    case fs::file_type::directory:  // 目录
    case fs::file_type::symlink:    // 符号链接
    case fs::file_type::character:  // 字符设备
    case fs::file_type::block:      // 块设备
    case fs::file_type::fifo:       // FIFO管道
    case fs::file_type::socket:     // 套接字
    case fs::file_type::unknown:    // 未知类型
    case fs::file_type::none:       // 文件不存在
}
```

### 权限 `fs::perms`
```cpp
fs::perms p = fs::status(path).permissions();

if ((p & fs::perms::owner_read) != fs::perms::none) {
    // 所有者有读权限
}

// 常用权限组合
fs::perms::all;        // 所有权限
fs::perms::owner_all;  // 所有者所有权限
```

## fs库主要定义了哪些类

C++17 的 `<filesystem>` 库主要定义了以下核心类，它们共同提供了完整的文件系统操作功能：

### 1. **`std::filesystem::path`** - 路径处理类
**最重要的类**，表示文件系统路径，提供路径的构建、解析和操作。

```cpp
namespace fs = std::filesystem;

// 构造和赋值
fs::path p1("/home/user/file.txt");
fs::path p2 = "C:\\Windows\\System32";
fs::path p3 = p1;

// 路径操作
p1.filename();    // "file.txt"
p1.parent_path(); // "/home/user"
p1.extension();   // ".txt"
p1 /= "subdir";   // 添加路径组件
```

### 2. **`std::filesystem::file_status`** - 文件状态类
包含文件的类型和权限信息。

```cpp
fs::file_status status = fs::status("/path/to/file");

// 获取文件类型和权限
fs::file_type type = status.type();
fs::perms permissions = status.permissions();
```

### 3. **`std::filesystem::directory_entry`** - 目录条目类
表示目录迭代器返回的条目，缓存了文件状态信息以提高性能。

```cpp
for (const auto& entry : fs::directory_iterator("/path")) {
    // entry 是 directory_entry 对象
    std::cout << "Path: " << entry.path() << std::endl;
    std::cout << "Is file: " << entry.is_regular_file() << std::endl;
    std::cout << "Size: " << entry.file_size() << std::endl;
}
```

### 4. **`std::filesystem::directory_iterator`** - 目录迭代器
用于遍历目录内容（非递归）。

```cpp
// 遍历目录
for (const auto& entry : fs::directory_iterator("/path/to/dir")) {
    std::cout << entry.path() << std::endl;
}
```

### 5. **`std::filesystem::recursive_directory_iterator`** - 递归目录迭代器
递归遍历目录及其所有子目录。

```cpp
// 递归遍历
for (const auto& entry : fs::recursive_directory_iterator("/path/to/dir")) {
    std::cout << entry.path() << std::endl;
    std::cout << "Depth: " << entry.depth() << std::endl; // 当前深度
}
```

### 6. **`std::filesystem::file_type`** - 文件类型枚举
表示不同类型的文件系统对象。

```cpp
enum class file_type {
    none,        // 不存在或无法确定
    not_found,   // 文件不存在
    regular,     // 普通文件
    directory,   // 目录
    symlink,     // 符号链接
    block,       // 块设备
    character,   // 字符设备
    fifo,        // FIFO管道
    socket,      // 套接字
    unknown      // 未知类型
};
```

### 7. **`std::filesystem::perms`** - 文件权限枚举
表示文件的访问权限。

```cpp
enum class perms {
    none = 0,
    
    // 所有者权限
    owner_read, owner_write, owner_exec,
    owner_all = owner_read | owner_write | owner_exec,
    
    // 组权限
    group_read, group_write, group_exec,
    group_all = group_read | group_write | group_exec,
    
    // 其他用户权限
    others_read, others_write, others_exec,
    others_all = others_read | others_write | others_exec,
    
    // 所有权限
    all = owner_all | group_all | others_all,
    
    // 特殊权限位
    set_uid, set_gid, sticky_bit,
    
    // 掩码
    mask = all | set_uid | set_gid | sticky_bit,
    
    // 未知权限
    unknown
};
```

### 8. **`std::filesystem::space_info`** - 磁盘空间信息类
表示文件系统的空间信息。

```cpp
fs::space_info info = fs::space("/path");

std::cout << "Total: " << info.capacity << " bytes" << std::endl;
std::cout << "Free: " << info.free << " bytes" << std::endl;
std::cout << "Available: " << info.available << " bytes" << std::endl;
```

### 9. **`std::filesystem::file_time_type`** - 文件时间类型
表示文件的时间戳（最后修改时间等）。

```cpp
fs::file_time_type last_write = fs::last_write_time("/path/to/file");

// 转换为time_t（需要C++20的clock_cast，或使用其他方法）
std::time_t c_time = decltype(last_write)::clock::to_time_t(last_write);
std::cout << "Last modified: " << std::ctime(&c_time);
```

### 10. **异常类**
### `std::filesystem::filesystem_error`
文件系统操作抛出的异常类，包含额外的路径信息。

```cpp
try {
    fs::file_size("nonexistent.txt");
} catch (const fs::filesystem_error& e) {
    std::cout << "Error: " << e.what() << std::endl;
    std::cout << "Path1: " << e.path1() << std::endl;
    std::cout << "Path2: " << e.path2() << std::endl; // 如果有第二个路径
}
```

## 类之间的关系和使用模式

### 典型工作流程

```cpp
#include <filesystem>
#include <iostream>

namespace fs = std::filesystem;

int main() {
    // 1. 使用 path 构建路径
    fs::path dir_path = "/home/user/documents";
    
    // 2. 检查路径状态
    if (fs::exists(dir_path) && fs::is_directory(dir_path)) {
        
        // 3. 使用 directory_iterator 遍历
        for (const auto& entry : fs::directory_iterator(dir_path)) {
            
            // 4. 每个 entry 是 directory_entry，包含缓存的状态
            if (entry.is_regular_file()) {
                
                // 5. 获取详细状态信息
                fs::file_status st = entry.status();
                
                std::cout << "File: " << entry.path().filename() << std::endl;
                std::cout << "Size: " << entry.file_size() << " bytes" << std::endl;
                std::cout << "Type: " << static_cast<int>(st.type()) << std::endl;
            }
        }
        
        // 6. 检查磁盘空间
        fs::space_info space = fs::space(dir_path);
        std::cout << "Free space: " << space.free / (1024*1024) << " MB" << std::endl;
    }
    
    return 0;
}
```

## 内存管理特点

这些类都遵循**值语义**（value semantics）：
- 可以安全地拷贝和赋值
- 使用RAII原则管理资源
- 大多数操作不会抛出异常（除非使用可能抛出的重载）

## 跨平台支持

`<filesystem>` 库自动处理不同操作系统的路径差异：
- Unix/Linux: `/` 作为路径分隔符
- Windows: `\` 作为路径分隔符，但也接受 `/`
- 自动处理驱动器字母（Windows）
- 自动处理根目录表示

这些类共同提供了一个现代、安全、易用的文件系统操作接口，完全替代了传统的C风格文件操作函数。
<!--stackedit_data:
eyJoaXN0b3J5IjpbNzUxMjUyMzQwXX0=
-->