# 花里胡哨的VIM

---

## 一、前期准备工作

1.VIM版本要为9.0以上，自行检查，如果不是的话，使用PPA安装
```bash
sudo add-apt-repository ppa:jonathonf/vim
sudo apt update#Ubuntu发行版会自动更新，不需要手动
sudo apt install vim
```

2.将系统中的 Node.js 升级到最新的 LTS 版本（通常推荐 v18 或 v20）
移除旧版本：
```bash
sudo apt remove nodejs
```

添加 NodeSource 源（安装 v20 LTS 版本）：
```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | bash -
```

安装新版本：
```bash
sudo apt install -y nodejs
```
>如果旧版没卸载干净可能会有报错，根据报错内容自行清理。

验证版本：
```bash
node -v
# 输出应该类似 v20.x.x
```

3.安装clangd
```bash
sudo apt install clang clangd
```

4.安装插件管理器 (Vim-Plug)
```bash
curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```

## 二、vim配置文件

~/.vimrc
```bash
vim9script
# ==========================================
# 0. 插件管理 (基于 vim-plug)
# ==========================================
call plug#begin('~/.vim/plugged')

# --- 核心大脑: LSP 与补全 ---
Plug 'neoclide/coc.nvim', {'branch': 'release'}

# --- C++ 增强 ---
# 既然用了 Vim9，我们用基于 Vim9 的高性能高亮插件，比旧版正则快得多
Plug 'bfrg/vim-cpp-modern'

# --- 界面美化 (颜值即正义) ---
# 顶栏 Tab 栏和底栏状态条
Plug 'vim-airline/vim-airline'
Plug 'vim-airline/vim-airline-themes'
# 经典的深色主题 (护眼)
#Plug 'sainnhe/gruvbox-material'
#仿VSCode主题
Plug 'tomasiser/vim-code-dark'
# 文件树图标 (必须安装 Nerd Font)
Plug 'ryanoasis/vim-devicons'

# --- 工程效率工具 (让手感像 IDE) ---
# 左侧文件树
Plug 'preservim/nerdtree'
# 自动成对补全括号/引号 (写代码必备)
Plug 'jiangmiao/auto-pairs'
# 快速注释 (gcc 注释当前行)
Plug 'tpope/vim-commentary'
# 包裹修改神器 (cs"' 把双引号改单引号)
Plug 'tpope/vim-surround'

# --- 现代交互 ---
# 模糊搜索 (像 VSCode 的 Ctrl+P)
Plug 'junegunn/fzf', { 'do': { -> fzf#install() } }
Plug 'junegunn/fzf.vim'
# 浮动终端 (按 F2 唤出，像 VSCode 的 Ctrl+`)
Plug 'voldikss/vim-floaterm'
# 调试器
Plug 'puremourning/vimspector'

call plug#end()

# ==========================================
# 1. 基础设置 (像工程师一样思考)
# ==========================================
set nocompatible
filetype plugin indent on
syntax on

# 开启真彩色支持 (必须，否则主题很丑)
 set termguicolors
# 开启鼠标支持 (偶尔点点很方便)
set mouse=a
# 显示行号和相对行号 (方便跳转)
set number relativenumber
# 高亮当前行
set cursorline
# 永远显示左侧标志列 (防止报错时屏幕抖动)
set signcolumn=yes
# 缩进设置 (C++ 标准 4 空格)
set expandtab
set tabstop=4
set shiftwidth=4
set softtabstop=4
set autoindent
set smartindent
# 搜索忽略大小写，除非包含大写
set ignorecase
set smartcase
# 很多现代插件需要的更新时间设置
set updatetime=100
# 也就是 Backspace 可以删掉任何东西
set backspace=indent,eol,start

# --- 主题设置 ---
# 设置背景为深色
set background=dark
colorscheme codedark
#g:airline_theme = 'codedark'
# 启用 Gruvbox Material 主题
#g:gruvbox_material_background = 'hard'
#colorscheme gruvbox-material
# 让 Airline 自动匹配主题
#g:airline_theme = 'gruvbox_material'
# 开启顶部 Tab 栏 (类似 VSCode 顶部的文件标签)
g:airline#extensions#tabline#enabled = 1
g:airline#extensions#tabline#formatter = 'unique_tail'
# 使用 Powerline 字体箭头
g:airline_powerline_fonts = 1

# ==========================================
# 2. 键位映射 (Space Leader 系统)
# ==========================================
# 将 Leader 键映射为空格 (现代 Vim 的标配，大拇指最方便)
g:mapleader = " "

# --- 常用快捷键 ---
# 快速保存 (空格 + w)
nnoremap <Leader>w :w<CR>
# 快速退出 (空格 + q)
nnoremap <Leader>q :q<CR>
# 清除搜索高亮 (空格 + l)
nnoremap <Leader>l :nohlsearch<CR>

# --- 窗口切换 (Ctrl + hjkl) ---
nnoremap <C-h> <C-w>h
nnoremap <C-j> <C-w>j
nnoremap <C-k> <C-w>k
nnoremap <C-l> <C-w>l

# --- 插件快捷键 ---
# 打开/关闭文件树 (空格 + e)
nnoremap <Leader>e :NERDTreeToggle<CR>
# 在当前文件定位文件树位置
nnoremap <Leader>f :NERDTreeFind<CR>

# 模糊搜索文件 (空格 + p, 习惯 VSCode 的 Cmd+P)
nnoremap <Leader>p :Files<CR>
# 搜索代码内容 (空格 + s)
nnoremap <Leader>s :Rg<CR>

# 浮动终端 (F2 开关)
nnoremap <F2> :FloatermToggle<CR>
tnoremap <F2> <C-\><C-n>:FloatermToggle<CR>

# ==========================================
# 3. Coc.nvim 深度配置 (IDE 的灵魂)
# ==========================================
# Tab 键选择补全
inoremap <silent><expr> <TAB>
      \ coc#pum#visible() ? coc#pum#next(1) :
      \ CheckBackspace() ? "\<Tab>" :
      \ coc#refresh()
inoremap <expr><S-TAB> coc#pum#visible() ? coc#pum#prev(1) : "\<C-h>"
inoremap <silent><expr> <CR> coc#pum#visible() ? coc#pum#confirm()
                              \: "\<C-g>u\<CR>\<c-r>=coc#on_enter()\<CR>"

function CheckBackspace()
  var col = col('.') - 1
  return !col || getline('.')[col - 1]  =~# '\s'
endfunction

# 代码导航 (gd 跳转定义, gr 查看引用, K 查看文档)
nmap <silent> gd <Plug>(coc-definition)
nmap <silent> gy <Plug>(coc-type-definition)
nmap <silent> gi <Plug>(coc-implementation)
nmap <silent> gr <Plug>(coc-references)
# K 键查看光标下函数的文档 (非常常用！)
nnoremap <silent> K :call ShowDocumentation()<CR>

function ShowDocumentation()
  if (index(['vim','help'], &filetype) >= 0)
    execute 'h '.expand('<cword>')
  elseif (coc#rpc#ready())
    call CocActionAsync('doHover')
  else
    execute '!' . &keywordprg . " " . expand('<cword>')
  endif
endfunction

# 代码重命名 (空格 + rn)
nmap <leader>rn <Plug>(coc-rename)

# 格式化代码 (空格 + fm)
nmap <leader>fm :call CocAction('format')<CR>

# ==========================================
# 4. Vimspector 调试配置
# ==========================================
nmap <F5> <Plug>VimspectorContinue
nmap <F9> <Plug>VimspectorToggleBreakpoint
nmap <F10> <Plug>VimspectorStepOver
nmap <F11> <Plug>VimspectorStepInto
```
文件中已经添加了仿vscode和gruvbox主题，可根据需要自行修改。

修改完配置文件后重新打开vim，运行
```
:PlugInstall
```
安装相应的插件

然后安装 Coc 扩展,继续在vim中运行
```
:CocInstall coc-clangd coc-cmake coc-json coc-sh coc-vimlsp
```

可选：安装调试器适配器
```
:VimspectorInstall vscode-cpptools
```

---


## 三、部分配置使用说明书


### 前言：核心操作逻辑

在这套配置中，所有的“魔法”都基于两个核心概念：

1. **Leader Key (空格键 Space)**：这是你的“指挥官键”。几乎所有自定义的高级功能都以 `<空格>` 开头。
* 按下 空格 + w = 保存 (Write)
* 按下 空格 + q = 退出 (Quit)
* 按下 空格 + l = 清除高亮 (Clear Highlight)

2. **模式切换**：
* **普通模式 (Normal)**：默认模式，用于移动、浏览、按快捷键。按 `Esc` 总是回到这里。
* **插入模式 (Insert)**：按 `i` 进入，像记事本一样打字。
* **可视模式 (Visual)**：按 `v` 进入，用于选中代码块。

---

### 第一章：项目管理与文件浏览

这部分解决“如何在成百上千个文件中快速穿梭”的问题。

#### 1. 文件资源管理器 (NERDTree)

**场景**：你需要查看项目目录结构，新建、删除或重命名文件。

* **打开/关闭侧边栏**：`<空格> + e`
* **在侧边栏中的操作**（光标在左侧文件树时）：
* `o` (小写)：打开光标下的文件。
* `s`：**分屏打开**（左右分屏），在对比代码时非常有用。
* `i`：**水平分屏打开**（上下分屏）。
* `C` (大写)：将当前选中的目录设置为“根目录”（进入子文件夹）。
* `u`：返回上一级目录。
* `I` (大写)：切换显示/隐藏文件（如 `.git` 或 `.o` 文件）。
* **`m` (关键功能)**：**唤出菜单**。按 `m` 后你会看到底部出现选项：
* 输入 `a`：**新建文件或文件夹**（输入名字以 `/` 结尾自动变成文件夹）。
* 输入 `d`：**删除**当前文件。
* 输入 `m`：**重命名/移动**文件。

#### 2. 模糊搜索神器 (FZF - Files)

**场景**：你知道文件名，想秒开，不希望在目录树里一层层点。

* **启动搜索**：`<空格> + p`
* **使用技巧**：
* **模糊匹配**：找 `driver/usb/usb_core.c`，只需输入 `drusbc` 即可匹配。
* **打开方式**：
* `Enter`：直接打开。
* `Ctrl + x`：在上下分屏中打开。
* `Ctrl + v`：在左右分屏中打开。
* `Ctrl + t`：在新标签页 (Tab) 中打开。

#### 3. 全局代码搜索 (RipGrep - Rg)

**场景**：你想找 `MyStruct` 结构体在哪些文件里被定义或使用了。

* **启动搜索**：`<空格> + s`
* **使用技巧**：
* 输入代码片段，列表会实时更新。
* 按 `Enter` 跳转到对应行。
* **高级技巧**：搜索完毕后，按 `Alt + a` 可以把所有搜索结果放入 QuickFix 列表，方便批量处理（配合 `:copen`）。

---

### 第二章：代码编写与智能辅助 (IDE 核心)

这部分由 **Coc.nvim** 提供支持，体验类似 VS Code。

#### 1. 智能补全

* **触发**：输入代码时自动弹出。
* **选择**：
* `Tab`：向下选择。
* `Shift + Tab`：向上选择。
* `Enter`：确认选中。


* **参数提示**：当你输入函数参数时（例如 `func(`），如果配置正确，Vim 会在悬浮窗提示当前应该是哪个参数（int a, string b...）。

#### 2. 代码导航 (跳转)

这是阅读大型 C++ 项目（如 Linux 内核或 BSP）时的神技。

* **跳转到定义 (Go Definition)**：`gd`
* 光标在 `func()` 上按 `gd`，直接跳到函数实现处。


* **跳转到引用 (Go References)**：`gr`
* 想看谁调用了这个函数？按 `gr`，会弹出一个列表显示所有调用点。


* **查看文档 (Hover)**：`K` (大写)
* 光标下的变量类型是什么？函数注释是什么？按 `K` 查看悬浮窗。


* **跳转历史 (Time Travel)**：
* **`Ctrl + o`**：**回到上一个位置**（比如你按 `gd` 跳走了，看一眼想回去，就按这个）。
* **`Ctrl + i`**：前进到下一个位置。

#### 3. 诊断与修复 (报错处理)

* **实时报错**：代码写错时，行号左边会出现红色 `>>` 或 `E` 标志，代码下方有波浪线。
* **查看报错详情**：
* 光标移到波浪线上，稍微停顿，会显示错误信息。
* 或者按 `[g` （跳转到上一个错误）和 `]g` （跳转到下一个错误）。


* **快速修复 (Quick Fix)**：
* 如果在报错行按 `<空格> + a` (CocAction)，如果有自动修复方案（如缺少头文件），它会提示你直接应用。

#### 4. 代码重构

* **重命名 (Rename)**：`<空格> + rn`
* 修改变量名，项目中所有引用该变量的地方都会自动修改（比简单的查找替换安全得多）。


* **格式化 (Format)**：`<空格> + fm`
* 调用 `clang-format` 自动排版当前文件。

---

### 第三章：极速编辑

这部分是 Vim 超越普通 IDE 的地方，主要依靠 `vim-surround` 和 `auto-pairs`。

#### 1. 包裹修改 (Surround) - 必须掌握

**口诀**：`c` (改变), `d` (删除), `y` (添加), `s` (Surround 包裹物)。

* **场景 A：把双引号改单引号**
* 代码：`"Hello World"`
* 操作：光标在引号内，按 `cs"'` (Change Surround " to ')
* 结果：`'Hello World'`

* **场景 B：删除括号**
* 代码：`(int a)`
* 操作：光标在括号内，按 `ds(` (Delete Surround ()
* 结果：`int a`

* **场景 C：给单词加括号/引号 (最常用)**
* 代码：`return x;` -> 想变成 `return (x);`
* 操作：光标在 `x` 上，按 `ysiw)` (You Surround Inner Word with ) )
* 结果：`return (x);`
* *注：如果按 `ysiw(` (左括号)，会在括号内加空格 `( x )`；按右括号则不加空格。*

#### 2. 快速注释

* **注释当前行**：`gcc`
* **注释多行**：
1. 按 `v` 进入可视模式。
2. 按 `j` 或 `k` 选中多行。
3. 按 `gc`。


* **解开注释**：重复上述操作即可。

#### 3. 括号补全

* 输入 `(` 自动补全 `)`，光标在中间。
* 输入 `{` 并回车，自动补全 `}` 并换行缩进（C++ 写函数时极爽）。

---

### 第四章：窗口与终端管理

#### 1. 窗口分屏与切换

**场景**：左边看头文件 `header.h`，右边写实现 `source.cpp`。

* **分屏操作**：
* `:vsp` (Vertical Split)：左右分屏。
* `:sp` (Split)：上下分屏。


* **窗口间移动光标**（已配置为快捷键）：
* `Ctrl + h`：去左边窗口。
* `Ctrl + l`：去右边窗口。
* `Ctrl + j`：去下边窗口。
* `Ctrl + k`：去上边窗口。


* **调整窗口大小**：
* `:vertical resize +5` (增加宽度)
* `:vertical resize -5` (减小宽度)
* 或者直接用鼠标拖动分界线（因为开启了 `set mouse=a`）。



#### 2. 浮动终端

**场景**：编译代码、Git 提交，不需要退出 Vim。

* **开关终端**：`F2`
* **使用逻辑**：
1. 按 `F2` 弹出终端。
2. 输入 `g++ main.cpp && ./a.out`。
3. 看完结果，再次按 `F2` 隐藏（后台保持运行）。


* **新建终端**：如果在浮动终端内想再开一个 Tab，通常命令是 `:FloatermNew`。

---

### 第五章：调试 (Vimspector)

调试流程完全模仿 VS Code。

#### 1. 准备工作 (每个项目只需一次)

在项目根目录创建 `.vimspector.json`。如果没有这个文件，按 F5 会提示输入配置。

#### 2. 调试快捷键

* **F9**：**打断点/取消断点**。
* **F5**：**启动调试 / 继续运行** (Continue)。
* **F3**：**停止调试**。
* **F4**：**重启调试**。
* **F10**：**单步跳过** (Step Over) - 不进入函数内部。
* **F11**：**单步进入** (Step Into) - 进入函数内部。
* **F12**：**单步跳出** (Step Out) - 执行完当前函数并返回。

#### 3. 调试界面交互

* **查看变量**：调试时，左侧窗口会显示 `Variables`。你可以像编辑文本一样，把光标移过去，按 `Enter` 展开结构体或类。
* **观察 (Watch)**：在变量窗口，你可以输入 `:VimspectorWatch my_var` 来单独监视某个变量。
* **鼠标悬停**：调试时，把鼠标放在代码里的变量上，会显示当前的值。

---

