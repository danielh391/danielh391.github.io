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

## 实用示例

### 1. 递归计算目录大小
```cpp
uintmax_t directory_size(const fs::path& path) {
    uintmax_t size = 0;
    for (const auto& entry : fs::recursive_directory_iterator(path)) {
        if (entry.is_regular_file()) {
            size += entry.file_size();
        }
    }
    return size;
}
```

### 2. 查找特定扩展名的文件
```cpp
std::vector<fs::path> find_files(const fs::path& dir, const std::string& ext) {
    std::vector<fs::path> result;
    for (const auto& entry : fs::directory_iterator(dir)) {
        if (entry.is_regular_file() && entry.path().extension() == ext) {
            result.push_back(entry.path());
        }
    }
    return result;
}
```

### 3. 创建备份文件
```cpp
void create_backup(const fs::path& file) {
    if (fs::exists(file)) {
        fs::path backup = file;
        backup += ".bak";
        fs::copy_file(file, backup, fs::copy_options::overwrite_existing);
    }
}
```

### 4. 跨平台路径处理
```cpp
fs::path create_path(const std::string& dir, const std::string& filename) {
    fs::path full_path = dir;
    full_path /= filename;  // 自动使用正确的路径分隔符
    return full_path;
}
```

## 错误处理

```cpp
try {
    fs::file_size("nonexistent.txt");
} catch (const fs::filesystem_error& e) {
    std::cout << "Error: " << e.what() << std::endl;
    std::cout << "Path: " << e.path1() << std::endl;
}

// 或者使用error_code避免异常
std::error_code ec;
uintmax_t size = fs::file_size("nonexistent.txt", ec);
if (ec) {
    std::cout << "Error: " << ec.message() << std::endl;
}
```

这些类和函数使得C++中的文件系统操作变得非常直观和强大，完全替代了传统的C风格文件操作函数。
<!--stackedit_data:
eyJoaXN0b3J5IjpbNTczMTQzMTE1XX0=
-->