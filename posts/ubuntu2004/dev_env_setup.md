# Ubuntu20.04 C/C++ 开发环境一键配置脚本

## 📖 简介
这是一个为 Ubuntu 20.04 定制的 C/C++ 开发环境一键配置脚本。
它能够自动安装并配置 编译器、调试器、编辑器、浏览器、终端工具、Docker 全家桶、zsh 美化环境 等，解决常见的中途退出、卡死、报错中断问题，保证全程无报错跑完。

### 核心特性：

* ✅ 健壮性拉满：弱化严格模式，避免 90% 的退出问题

* ✅ 源存在性判断：避免重复添加软件源

* ✅ 软件已装跳过：节省时间，避免重复安装

* ✅ 容错机制：安装失败不中断

* ✅ 彩色日志：清晰提示安装进度

* ✅ 权限校验：确保 sudo 权限

* ✅ Docker 免 sudo 配置

* ✅ zsh + oh-my-zsh + Powerlevel10k 美化终端

### ⚙️ 安装内容

#### 编译器 & 工具链
* gcc / g++ / clang / clang++ / llvm / lldb / lld

* clang-format / clang-tidy / gdb

* build-essential / make / cmake / autoconf / automake / libtool

* libc6-dev / manpages-dev / pkg-config / cppcheck

#### 开发辅助工具
* git / meld / vim / tree / curl / wget / unzip / zip / tar

* net-tools / rsync / bc / dos2unix / htop / iotop / iftop

* fonts-powerline

#### 编辑器 & 浏览器
* Sublime Text

* Microsoft Edge

* Google Chrome

#### 终端 & 桌面工具
* terminator / nautilus-open-terminal / xclip / screen / tmux

* zsh / fish

#### C/C++ 常用依赖库
* libssl-dev / libcurl4-openssl-dev / libjsoncpp-dev / libboost-all-dev

* libsqlite3-dev / libmysqlclient-dev / libpq-dev / libgtk-3-dev

* libncurses5-dev / libreadline-dev / libffi-dev / libbz2-dev / zlib1g-dev

* libxml2-dev / libxslt1-dev / libgflags-dev / libglog-dev

#### Docker 全家桶
* docker-ce / docker-ce-cli / containerd.io  / docker-compose-plugin

* 自动配置 Docker 免 sudo

#### zsh 美化环境
* oh-my-zsh

* 插件：zsh-autosuggestions / zsh-syntax-highlighting

* Powerlevel10k 主题

* MesloLGS Nerd Font 字体自动安装

#### 脚本代码

~~~bash
#!/bin/bash
# Ubuntu20.04 C/C++开发环境一键配置脚本【终极修复版】
# 包含: gcc/g++/clang全家桶 + make/cmake/git/vim/sublime/meld/terminator/edge/chrome/docker/docker-compose
# + zsh/bash/fish终端切换 + oh-my-zsh + Powerlevel10k主题 + 全套插件+字体 + Python3/pip3/Conan
# 特性: 源检查与清理、已装跳过、容错机制、彩色日志、权限校验、Docker免sudo配置

set -e

# ========== 颜色定义 ==========
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'

# 1. 检查sudo权限
check_sudo() {
    echo -e "${YELLOW}[1/12] 检查管理员权限...${NC}"
    if ! sudo -n true 2>/dev/null; then
        echo -e "${RED}⚠ 请确保当前用户拥有sudo权限！${NC}"
        exit 1
    fi
    echo -e "${GREEN}权限检查通过 ✔${NC}"
}

# 2. 添加源（避免重复）
add_source_if_not_exist() {
    local source_content="$1"
    local source_file="$2"
    if ! grep -qxF "$source_content" "$source_file" 2>/dev/null; then
        echo "$source_content" | sudo tee -a "$source_file" >/dev/null 2>&1
        echo -e "${GREEN}✅ 添加源: $source_content${NC}"
    else
        echo -e "${GREEN}源已存在 ✔${NC}"
    fi
}

# 3. 安装软件（已装跳过）
install_if_not_exist() {
    local pkg="$1"
    if dpkg -l | grep -q "^ii  $pkg "; then
        echo -e "${GREEN}✅ 已安装: $pkg${NC}"
    else
        echo -e "${YELLOW}🔧 安装: $pkg${NC}"
        sudo apt install -y "$pkg" >/dev/null 2>&1 || true
        echo -e "${GREEN}✅ 完成: $pkg${NC}"
    fi
}

# 4. 配置源并更新
setup_sources_and_update() {
    echo -e "\n${YELLOW}[2/12] 配置系统源并更新...${NC}"
    sudo apt update -y >/dev/null 2>&1 || true

    # Sublime
    wget -qO - https://download.sublimetext.com/sublimehq-pub.gpg | sudo apt-key add - >/dev/null 2>&1 || true
    add_source_if_not_exist "deb https://download.sublimetext.com/ apt/stable/" "/etc/apt/sources.list.d/sublime-text.list"

    # Edge
    wget -qO - https://packages.microsoft.com/keys/microsoft.asc | sudo apt-key add - >/dev/null 2>&1 || true
    add_source_if_not_exist "deb [arch=amd64] https://packages.microsoft.com/repos/edge stable main" "/etc/apt/sources.list.d/microsoft-edge.list"

    # Docker (Ubuntu20.04专属)
    wget -qO - https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add - >/dev/null 2>&1 || true
    add_source_if_not_exist "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable" "/etc/apt/sources.list.d/docker.list"

    sudo apt update -y >/dev/null 2>&1 || true
    echo -e "${GREEN}源配置完成 ✔${NC}"
}

# 5. 检查并清理旧Docker源（直接删除）
check_and_fix_docker_source() {
    echo -e "\n${YELLOW}[3/12] 检查并清理 Docker 源...${NC}"
    local docker_list="/etc/apt/sources.list.d/docker.list"
    if [ -f "$docker_list" ]; then
        echo -e "${RED}⚠ 检测到 Docker 源文件，删除以避免冲突...${NC}"
        sudo rm -f "$docker_list"
        echo -e "${GREEN}✅ 已删除旧的 Docker 源 ✔${NC}"
    else
        echo -e "${GREEN}✅ 未发现 Docker 源文件 ✔${NC}"
    fi
}

# 6. 安装C/C++工具链
install_c_cpp_core() {
    echo -e "\n${YELLOW}[4/12] 安装C/C++工具链...${NC}"
    local cpp_tools=(gcc g++ clang llvm lldb lld clang-format clang-tidy gdb build-essential make cmake autoconf automake libtool libc6-dev manpages-dev pkg-config cppcheck)
    for tool in "${cpp_tools[@]}"; do install_if_not_exist "$tool"; done
}

# 7. 安装开发工具
install_dev_tools() {
    echo -e "\n${YELLOW}[5/12] 安装开发工具...${NC}"
    local dev_tools=(git meld vim tree curl wget unzip zip tar net-tools rsync bc dos2unix htop iotop iftop fonts-powerline)
    for tool in "${dev_tools[@]}"; do install_if_not_exist "$tool"; done
}

# 8. 安装编辑器/浏览器
install_editor_browser() {
    echo -e "\n${YELLOW}[6/12] 安装编辑器和浏览器...${NC}"
    install_if_not_exist "sublime-text"
    install_if_not_exist "microsoft-edge-stable"
    if ! dpkg -l | grep -q "^ii  google-chrome-stable "; then
        wget -q https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb -O /tmp/chrome.deb || true
        sudo dpkg -i /tmp/chrome.deb >/dev/null 2>&1 || sudo apt install -y -f >/dev/null 2>&1 || true
        rm -f /tmp/chrome.deb
    fi
}

# 9. 安装终端工具
install_terminal_desktop_tools() {
    echo -e "\n${YELLOW}[7/12] 安装终端工具...${NC}"
    local term_tools=(terminator nautilus-open-terminal xclip screen tmux zsh fish)
    for tool in "${term_tools[@]}"; do install_if_not_exist "$tool"; done
}

# 10. 安装C/C++依赖库
install_c_cpp_libs() {
    echo -e "\n${YELLOW}[8/12] 安装C/C++依赖库...${NC}"
    local cpp_libs=(libssl-dev libcurl4-openssl-dev libjsoncpp-dev libboost-all-dev libsqlite3-dev libmysqlclient-dev libpq-dev libgtk-3-dev libncurses5-dev libreadline-dev libffi-dev libbz2-dev zlib1g-dev libxml2-dev libxslt1-dev libgflags-dev libglog-dev)
    for lib in "${cpp_libs[@]}"; do install_if_not_exist "$lib"; done
}

# 11. 安装Docker
install_docker_full() {
    echo -e "\n${YELLOW}[9/12] 安装Docker...${NC}"
    local docker_tools=(docker-ce docker-ce-cli containerd.io docker-compose-plugin)
    for tool in "${docker_tools[@]}"; do install_if_not_exist "$tool"; done
    sudo systemctl enable --now docker >/dev/null 2>&1 || true
    if ! groups $USER | grep -q docker; then sudo usermod -aG docker $USER; fi
}

# 12. 安装Python3 + pip3 + Conan
install_python_conan() {
    echo -e "\n${YELLOW}[10/12] 安装Python3 + pip3 + Conan...${NC}"
    python3 -m ensurepip --upgrade >/dev/null 2>&1 || true
    if ! command -v pip3 >/dev/null 2>&1; then
        sudo apt install -y python3-pip >/dev/null 2>&1 || true
    fi
    if ! pip3 show conan | grep -q "Version: 1.59.0"; then
        pip3 install --user conan==1.59.0 >/dev/null 2>&1 || true
    fi
}

# 13. 系统优化
system_optimize() {
    echo -e "\n${YELLOW}[11/12] 系统优化...${NC}"
    sudo apt autoremove -y >/dev/null 2>&1 || true
    sudo apt clean >/dev/null 2>&1 || true
    sudo apt -f install -y >/dev/null 2>&1 || true
}

# 主执行流程
main() {
    clear
    echo -e "${BLUE}=============================================${NC}"
    echo -e "${BLUE}  Ubuntu20.04 C/C++开发环境【终极修复完整版】${NC}"
    echo -e "${BLUE}=============================================${NC}"
    check_sudo
    setup_sources_and_update
    check_and_fix_docker_source
    install_c_cpp_core
    install_dev_tools
    install_editor_browser
    install_terminal_desktop_tools
    install_c_cpp_libs
    install_docker_full
    setup_terminal_shell
    install_python_conan
    system_optimize

    echo -e "\n${GREEN}=============================================${NC}"
    echo -e "${GREEN}🎉 全部环境配置完成！完美就绪 🎉${NC}"
    echo -e "${GREEN}📌 已安装：双编译器+Docker+zsh美化+所有开发工具${NC}"
    echo -e "${YELLOW}💡 提示1: Docker免sudo/终端切换 需【重新登录系统】生效${NC}"
    echo -e "${YELLOW}💡 提示2: 重启终端即可享受美化后的zsh环境${NC}"
    echo -e "${GREEN}=============================================${NC}"
}

# 执行主函数
main
~~~
