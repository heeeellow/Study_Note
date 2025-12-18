# Linux tar 命令

---

## 1.分卷压缩与解压

Linux 下最通用的分卷压缩方式是结合 `tar`（打包/压缩）与 `split`（切割）命令使用管道流处理。

### 1.1 分卷压缩

将目录打包并分割成指定大小的多个文件。

```bash
# 语法
tar -czvf - <要打包的目录> | split -b <分卷大小> -d - <输出文件前缀>

# 示例：将 logs 目录压缩，每卷 500MB，命名为 logs.tar.gz.00, logs.tar.gz.01...
tar -czvf - logs/ | split -b 500M -d - logs.tar.gz.
```

**参数解析：**

  * `tar -czvf -`: 最后面的 `-` 代表输出到标准输出（Stdout），不写入磁盘文件。
  * `split -b 500M`: 设置切割大小（单位：`k`, `M`, `G`）。
  * `split -d`: 使用数字后缀（00, 01...）而非默认的字母（aa, ab...）。
  * `split -`: 最后面的 `-` 代表从标准输入（Stdin）读取数据。

### 1.2 分卷解压

将多个分卷文件合并还原。

```bash
# 语法
cat <分卷前缀>* | tar -xzvf -

# 示例：解压上述 logs.tar.gz.00 等文件
cat logs.tar.gz.* | tar -xzvf -
```

**原理：** `cat` 利用通配符读取所有分卷流，通过管道传回给 `tar` 进行解压。

-----

## 2. 常用基础命令速查

### 2.1 压缩 (打包)

根据需求选择不同的压缩算法（压缩率：xz \> bzip2 \> gzip；速度则反之）。

  * **`.tar.gz` (最常用，速度快)**
    ```bash
    tar -czvf archive.tar.gz /path/to/folder
    ```
  * **`.tar.bz2` (压缩率较高)**
    ```bash
    tar -cjvf archive.tar.bz2 /path/to/folder
    ```
  * **`.tar.xz` (压缩率最高，耗时久)**
    ```bash
    tar -cJvf archive.tar.xz /path/to/folder
    ```

### 2.2 解压 (解包)

现代 `tar` 通常能自动识别压缩格式，只需记住 `-xvf` 即可，但显式指定算法更保险。

  * **解压到当前目录**
    ```bash
    tar -xzvf archive.tar.gz
    ```
  * **解压到指定目录 (`-C` 参数)**
    ```bash
    # 目标目录必须先存在
    mkdir -p /opt/backup
    tar -xzvf archive.tar.gz -C /opt/backup
    ```

### 2.3 查看内容 (不解压)

在解压大文件前，先查看里面有什么。

```bash
tar -tvf archive.tar.gz
```

-----

## 3. 参数详解表

| 参数 | 含义 | 备注 |
| :--- | :--- | :--- |
| **-c** | **Create** (创建) | 建立新的归档文件 |
| **-x** | **Extract** (提取) | 从归档文件中提取文件 |
| **-t** | **List** (列表) | 查看归档文件内容 |
| **-v** | **Verbose** (详细) | 显示处理过程（推荐使用） |
| **-f** | **File** (文件) | 指定归档文件名（**必须是最后一个参数**，后接文件名） |
| **-C** | **Change Directory** | 切换到指定目录进行解压 |
| **-z** | **Gzip** | 处理 `.tar.gz` 格式 |
| **-j** | **Bzip2** | 处理 `.tar.bz2` 格式 |
| **-J** | **Xz** | 处理 `.tar.xz` 格式 |

-----

## 4. 进阶技巧

### 4.1 排除特定文件/文件夹 (`--exclude`)

打包项目源码时，通常需要排除 `.git` 目录或编译生成文件。

```bash
# 注意：--exclude pattern 最好放在命令靠前位置，且不加前导斜杠
tar --exclude='.git' --exclude='*.o' -czvf project.tar.gz ./project_src
```

### 4.2 仅打包，不压缩

如果你只是想把一堆文件变成一个包（方便传输），但不消耗 CPU 压缩：

```bash
tar -cvf bundle.tar /path/to/files
```

### 4.3 相对路径 vs 绝对路径

**警告：** 尽量**不要**使用绝对路径（如 `/home/user/data`）打包。

  * **坏处**：解压时会强制覆盖绝对路径下的文件，可能导致系统文件损坏或权限问题。
  * **做法**：先 `cd` 到父目录，使用相对路径打包。

<!-- end list -->

```bash
# 推荐做法
cd /home/user/
tar -czvf data.tar.gz ./data
```