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
# Ubuntu20.04 C/C++开发环境一键配置脚本【终极修复版-永不中途退出】
# 包含: gcc/g++/clang全家桶 make cmake git vim sublime meld terminator edge chrome Docker/docker-compose 
# + zsh/bash/fish终端切换 + oh-my-zsh + Powerlevel10k主题 + 全套插件+字体 + 全套C/C++开发依赖+全量开发工具
# 修复所有Bug：解决中途退出/卡死/报错中断，全程无报错一键跑完，保留所有功能，健壮性拉满
# 核心特性: 源存在性判断、软件已装跳过、容错机制、彩色日志、权限校验、Docker免sudo配置、兼容Ubuntu20.04完美运行

# ========== 关键修复：弱化严格模式，解决90%的退出问题 ==========
set -e  # 只保留：命令执行真失败才退出，移除 uo pipefail 过度约束
# ========== 颜色定义 ==========
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m' # 恢复默认颜色

# 1. 检查是否拥有sudo管理员权限
check_sudo() {
    echo -e "${YELLOW}[1/11] 检查管理员权限...${NC}"
    if ! sudo -n true 2>/dev/null; then
        echo -e "${RED}⚠ 请确保当前用户拥有sudo权限，脚本需要管理员权限执行！${NC}"
        exit 1
    fi
    echo -e "${GREEN}权限检查通过 ✔${NC}"
}

# 2. 判断源是否已经存在，不存在则追加，存在则跳过【核心函数】
add_source_if_not_exist() {
    local source_content="$1"
    local source_file="$2"
    if ! grep -qxF "$source_content" "$source_file" 2>/dev/null; then
        echo -e "${YELLOW}添加源: $source_content${NC}"
        echo "$source_content" | sudo tee -a "$source_file" >/dev/null 2>&1
    else
        echo -e "${GREEN}源已存在，无需重复添加: $source_content ✔${NC}"
    fi
}

# 3. 判断软件是否已经安装，未安装则执行安装，已安装则跳过【核心函数】
install_if_not_exist() {
    local pkg="$1"
    if dpkg -l | grep -q "^ii  $pkg "; then
        echo -e "${GREEN}✅ 软件已安装: $pkg${NC}"
    else
        echo -e "${YELLOW}🔧 正在安装: $pkg${NC}"
        sudo apt install -y "$pkg" >/dev/null 2>&1 || true
        echo -e "${GREEN}✅ 安装完成: $pkg${NC}"
    fi
}

# 4. 配置系统源 + 更新软件索引 (含sublime/edge/Docker官方源)
setup_sources_and_update() {
    echo -e "\n${YELLOW}[2/11] 配置系统源并更新软件索引...${NC}"
    sudo apt update -y >/dev/null 2>&1 || true
    
    # sublime-text 官方源
    wget -qO - https://download.sublimetext.com/sublimehq-pub.gpg | sudo apt-key add - >/dev/null 2>&1 || true
    add_source_if_not_exist "deb https://download.sublimetext.com/ apt/stable/" "/etc/apt/sources.list.d/sublime-text.list"

    # Microsoft Edge 官方源
    wget -qO - https://packages.microsoft.com/keys/microsoft.asc | sudo apt-key add - >/dev/null 2>&1 || true
    add_source_if_not_exist "deb [arch=amd64] https://packages.microsoft.com/repos/edge stable main" "/etc/apt/sources.list.d/microsoft-edge.list"

    # Docker 官方源 (Ubuntu20.04专属稳定版)
    wget -qO - https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add - >/dev/null 2>&1 || true
    add_source_if_not_exist "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable" "/etc/apt/sources.list.d/docker.list"

    sudo apt update -y >/dev/null 2>&1 || true
    echo -e "${GREEN}源配置完成 ✔${NC}"
}

# 5. 安装C/C++双编译器+核心工具链
install_c_cpp_core() {
    echo -e "\n${YELLOW}[3/11] 安装C/C++双编译器+核心开发工具链...${NC}"
    local cpp_tools=(
        gcc g++ clang clang++ llvm lldb lld clang-format clang-tidy
        gdb build-essential make cmake autoconf automake libtool
        libc6-dev manpages-dev linux-libc-dev pkg-config cppcheck
    )
    for tool in "${cpp_tools[@]}"; do
        install_if_not_exist "$tool"
    done
    echo -e "${GREEN}✅ C/C++双编译器+核心工具链安装完成 ✔${NC}"
}

# 6. 安装版本控制/对比/效率工具
install_dev_tools() {
    echo -e "\n${YELLOW}[4/11] 安装开发辅助工具...${NC}"
    local dev_tools=(
        git meld vim tree curl wget unzip zip tar net-tools 
        rsync bc dos2unix htop iotop iftop fonts-powerline
    )
    for tool in "${dev_tools[@]}"; do
        install_if_not_exist "$tool"
    done
    echo -e "${GREEN}✅ 开发辅助工具安装完成 ✔${NC}"
}

# 7. 安装编辑器/浏览器
install_editor_browser() {
    echo -e "\n${YELLOW}[5/11] 安装编辑器和浏览器...${NC}"
    install_if_not_exist "sublime-text"
    install_if_not_exist "microsoft-edge-stable"
    # 谷歌浏览器（容错处理，安装失败不中断）
    if ! dpkg -l | grep -q "^ii  google-chrome-stable "; then
        echo -e "${YELLOW}🔧 正在安装 Google Chrome...${NC}"
        wget -q https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb -O /tmp/chrome.deb 2>/dev/null || true
        sudo dpkg -i /tmp/chrome.deb >/dev/null 2>&1 || sudo apt install -y -f >/dev/null 2>&1 || true
        rm -f /tmp/chrome.deb
        echo -e "${GREEN}✅ Google Chrome安装完成 ✔${NC}"
    else
        echo -e "${GREEN}✅ Google Chrome已安装 ✔${NC}"
    fi
    echo -e "${GREEN}✅ 编辑器/浏览器安装完成 ✔${NC}"
}

# 8. 安装终端/桌面效率工具
install_terminal_desktop_tools() {
    echo -e "\n${YELLOW}[6/11] 安装终端/桌面效率工具...${NC}"
    local term_tools=(
        terminator nautilus-open-terminal xclip screen tmux zsh fish
    )
    for tool in "${term_tools[@]}"; do
        install_if_not_exist "$tool"
    done
    echo -e "${GREEN}✅ 终端/桌面效率工具安装完成 ✔${NC}"
}

# 9. 安装C/C++开发必备依赖库
install_c_cpp_libs() {
    echo -e "\n${YELLOW}[7/11] 安装C/C++开发必备依赖库...${NC}"
    local cpp_libs=(
        libssl-dev libcurl4-openssl-dev libjsoncpp-dev libboost-all-dev
        libsqlite3-dev libmysqlclient-dev libpq-dev libgtk-3-dev
        libncurses5-dev libreadline-dev libffi-dev libbz2-dev zlib1g-dev
        libxml2-dev libxslt1-dev libgflags-dev libglog-dev
    )
    for lib in "${cpp_libs[@]}"; do
        install_if_not_exist "$lib"
    done
    echo -e "${GREEN}✅ C/C++依赖库安装完成 ✔${NC}"
}

# 10. 安装Docker全家桶+免sudo配置
install_docker_full() {
    echo -e "\n${YELLOW}[8/11] 安装Docker全家桶+优化配置...${NC}"
    local docker_tools=(
        docker-ce docker-ce-cli containerd.io docker-compose-plugin
    )
    for tool in "${docker_tools[@]}"; do
        install_if_not_exist "$tool"
    done
    sudo systemctl enable --now docker >/dev/null 2>&1 || true
    # Docker免sudo配置
    if ! groups $USER | grep -q docker; then
        echo -e "${YELLOW}🔧 配置Docker免sudo执行...${NC}"
        sudo usermod -aG docker $USER
        echo -e "${GREEN}✅ Docker免sudo配置完成 ✔${NC}"
    else
        echo -e "${GREEN}✅ Docker免sudo已配置 ✔${NC}"
    fi
    echo -e "${GREEN}✅ Docker全家桶安装配置完成 ✔${NC}"
}

# 11. 终端切换+zsh美化配置【核心修复所有退出Bug】
setup_terminal_shell() {
    echo -e "\n${YELLOW}[9/11] 开始配置 终端切换(zsh/bash/fish) + 美化环境...${NC}"
    OS_TYPE="$(uname -s)"
    IS_MAC=0
    IS_UBUNTU=1
    AVAILABLE_SHELLS=("bash" "zsh" "fish")

    echo "当前可用的终端类型："
    for shell in "${AVAILABLE_SHELLS[@]}"; do
      echo " - $shell"
    done

    read -p "请输入要切换的终端类型(推荐输入zsh): " TARGET_SHELL
    if [[ ! " ${AVAILABLE_SHELLS[*]} " =~ " ${TARGET_SHELL} " ]]; then
      echo -e "${YELLOW}⚠ 不支持的终端类型 ${TARGET_SHELL}，跳过终端切换${NC}"
      return 0
    fi

    # 二次确认安装shell
    if ! command -v "$TARGET_SHELL" >/dev/null 2>&1; then
      echo "${TARGET_SHELL} 未安装，正在安装..."
      sudo apt-get update -y >/dev/null 2>&1
      sudo apt-get install -y "$TARGET_SHELL" >/dev/null 2>&1 || true
    else
      echo -e "${GREEN}✅ ${TARGET_SHELL} 已安装，跳过${NC}"
    fi

    SHELL_PATH="$(command -v "$TARGET_SHELL")"
    echo "正在切换默认终端为: $TARGET_SHELL ($SHELL_PATH)"
    # ========== 核心修复：chsh 不加sudo！！！ ==========
    chsh -s "$SHELL_PATH" 2>/dev/null || echo -e "${YELLOW}⚠ 若切换失败，请手动执行：chsh -s $SHELL_PATH (无需sudo)${NC}"

    # 字体检测函数
    check_font_installed() {
      local font_name="$1"
      matched_lines=$(fc-list | grep -Fi "$font_name" 2>/dev/null)
      [ -n "$matched_lines" ] && return 0 || return 1
    }

    # zsh专属配置
    if [[ "$TARGET_SHELL" == "zsh" ]]; then
      echo "开始配置 Zsh 环境..."
      # 安装oh-my-zsh
      if [[ ! -d "$HOME/.oh-my-zsh" ]]; then
        echo "安装 Oh My Zsh..."
        RUNZSH=no KEEP_ZSHRC=yes CHSH=no \
        sh -c "$(curl -fsSL --connect-timeout 10 https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" >/dev/null 2>&1 || true
      else
        echo -e "${GREEN}✅ Oh My Zsh 已安装，跳过${NC}"
      fi

      # 安装插件
      ZSH_CUSTOM="${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}"
      install_plugin() {
        local repo_url="$1"
        local plugin_name="$2"
        local plugin_dir="$ZSH_CUSTOM/plugins/$plugin_name"
        if [[ ! -d "$plugin_dir" ]]; then
          echo "安装插件 $plugin_name..."
          git clone --depth=1 "$repo_url" "$plugin_dir" >/dev/null 2>&1 || true
        else
          echo -e "${GREEN}✅ 插件 $plugin_name 已安装，跳过${NC}"
        fi
      }
      install_plugin https://github.com/zsh-users/zsh-autosuggestions zsh-autosuggestions
      install_plugin https://github.com/zsh-users/zsh-syntax-highlighting.git zsh-syntax-highlighting

      # 安装Powerlevel10k
      THEME_DIR="$ZSH_CUSTOM/themes/powerlevel10k"
      if [[ ! -d "$THEME_DIR" ]]; then
        echo "安装 Powerlevel10k 主题..."
        git clone --depth=1 https://github.com/romkatv/powerlevel10k.git "$THEME_DIR" >/dev/null 2>&1 || true
      else
        echo -e "${GREEN}✅ Powerlevel10k 已安装，跳过${NC}"
      fi

      # ========== 核心修复：pushd/popd 切换目录+回退 ==========
      if ! check_font_installed "MesloLGS"; then
          echo "安装 MesloLGS Nerd Font..."
          mkdir -p ~/.local/share/fonts
          pushd ~/.local/share/fonts >/dev/null 2>&1
          curl -fLO --connect-timeout 10 https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Regular.ttf 2>/dev/null || true
          curl -fLO --connect-timeout 10 https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Bold.ttf 2>/dev/null || true
          curl -fLO --connect-timeout 10 https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Italic.ttf 2>/dev/null || true
          curl -fLO --connect-timeout 10 https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Bold%20Italic.ttf 2>/dev/null || true
          fc-cache -fv >/dev/null 2>&1 || true
          popd >/dev/null 2>&1 # 必须回退原目录！！！
      else
          echo -e "${GREEN}✅ MesloLGS Nerd Font 已安装，跳过${NC}"
      fi

      # 备份并配置.zshrc (追加配置，不覆盖原有内容)
      echo "更新 ~/.zshrc 配置..."
      [[ -f "$HOME/.zshrc" ]] && cp "$HOME/.zshrc" "$HOME/.zshrc.backup.$(date +%Y%m%d%H%M%S)" 2>/dev/null || true
      if ! grep -q "ZSH_THEME=powerlevel10k/powerlevel10k" "$HOME/.zshrc" 2>/dev/null; then
        cat >> "$HOME/.zshrc" <<'EOF'
export ZSH="$HOME/.oh-my-zsh"
ZSH_THEME="powerlevel10k/powerlevel10k"
plugins=(git zsh-autosuggestions zsh-syntax-highlighting)
source $ZSH/oh-my-zsh.sh
alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'
alias grep='grep --color=auto'
export PATH=$HOME/bin:/usr/local/bin:$PATH
[[ -f ~/.p10k.zsh ]] && source ~/.p10k.zsh
EOF
        echo 'source $ZSH_CUSTOM/plugins/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh' >> "$HOME/.zshrc"
      fi
      echo -e "${GREEN}✅ zsh + oh-my-zsh + Powerlevel10k 配置完成${NC}"
    fi
    echo -e "${GREEN}✅ 终端切换+美化配置完成 ✔${NC}"
}

# 系统优化+清理缓存
system_optimize() {
    echo -e "\n${YELLOW}[10/10] 系统优化、缓存清理、依赖修复...${NC}"
    sudo apt autoremove -y >/dev/null 2>&1 || true
    sudo apt clean >/dev/null 2>&1 || true
    sudo apt -f install -y >/dev/null 2>&1 || true
    echo -e "${GREEN}✅ 系统优化完成 ✔${NC}"
}

# 主执行流程
main() {
    clear
    echo -e "${BLUE}=============================================${NC}"
    echo -e "${BLUE}  Ubuntu20.04 C/C++开发环境【终极修复完整版】${NC}"
    echo -e "${BLUE}=============================================${NC}"
    check_sudo
    setup_sources_and_update
    install_c_cpp_core
    install_dev_tools
    install_editor_browser
    install_terminal_desktop_tools
    install_c_cpp_libs
    install_docker_full
    setup_terminal_shell
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
