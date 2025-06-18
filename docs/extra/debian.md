---
title: Настройка Debian для разработки
tags:
    - Linux
    - Workflow
hide:
    - tags
    - navigation
---
# ⚙️ Настройка Debian для разработки

Этот гайд поможет подготовить Debian-систему для **комфортной** и **продуктивной** разработки, включая установку необходимых пакетов, настройку графического окружения и полезных сервисов.

---

## 📦 Настройка репозиториев Debian

### 🐧 **Nala** - удобная замена для apt

Удобный менеджер пакетов с красивым выводом:

```bash
sudo apt install nala
```

### 🔄 Переключение на SID (testing) ветку

Чтобы использовать последние версии пакетов, перейдите на SID:

```bash
sudo nano /etc/apt/sources.list
```

В файле замените все упоминания `bookworm` (или другого релиза) на `sid`.

Обновите список пакетов и систему:

```bash
sudo nala update && sudo nala upgrade
```

### 🛠️ Подключение 32-битной архитектуры

Для поддержки 32-битных приложений, например Wine:

```bash
sudo dpkg --add-architecture i386
sudo nala update
```

### 🔄  Перезапуск системы

После обновлений рекомендуем перезагрузиться:

```bash
sudo shutdown -r now
```

---

## 🛠️ Установка основных утилит

### ⚙️ Установка Zsh и смена оболочки

Установка `zsh`:

```bash
sudo nala install zsh
chsh -s $(which zsh)
```

Установка менеджера плагинов для Zsh:

```bash
zsh <(curl -s https://raw.githubusercontent.com/zap-zsh/zap/master/install.zsh) \
  --branch release-v1 --keep
```

### 🔧 Системные библиотеки и инструменты сборки

Необходимы для компиляции и установки различных пакетов, особенно через `cargo`:

```bash
sudo nala install -y \
    build-essential \
    pkg-config \
    uidmap \
    libssl-dev \
    libudev-dev \
    libxcb-shape0-dev \
    libxcb-xfixes0-dev \
    libxkbcommon-dev \
    libc6 \
    libgcc-s1 \
    libgssapi-krb5-2 \
    libicu72 \
    libssl3 \
    libstdc++6 \
    zlib1g \
    unzip \
    zip \
    gnupg2 \
    git \
    curl
```

### 💻 Утилиты командной строки и интерфейса

Полезные инструменты для повседневной работы и диагностики:

```bash
sudo nala install -y \
    tmux \
    neovim \
    direnv \
    git-delta \
    stow \
    zoxide \
    starship \
    trash-cli \
    eza \
    bat \
    fzf \
    ripgrep \
    sd \
    fastfetch \
    cpufetch \
    procs \
    du-dust \
    htop \
    glances \
    xh \
    jq
```

| Утилита       | Описание                                                                  |
| ------------- | ------------------------------------------------------------------------- |
| **tmux**      | Мультиплексор терминала: управление несколькими сессиями в одном окне     |
| **neovim**    | Современный форк Vim с расширенной поддержкой плагинов и скриптов         |
| **direnv**    | Автоматическая подгрузка переменных окружения из `.envrc`                 |
| **git-delta** | Красивый и читаемый вывод `git diff` и `git blame`                        |
| **stow**      | Управление dotfiles с помощью символических ссылок                        |
| **zoxide**    | Улучшенная замена `cd`, запоминает наиболее часто используемые директории |
| **starship**  | Кроссплатформенный быстрый prompt для Zsh/Bash/Fish                       |
| **trash-cli** | Команда `trash` вместо `rm` — перемещает файлы в корзину                  |
| **eza**       | Современная альтернатива `ls` с цветами, иконками и деревом               |
| **bat**       | Подсветка синтаксиса и просмотр файлов как `cat`, но красивее             |
| **fzf**       | Мгновенный fuzzy-поиск по файлам, истории, командам                       |
| **ripgrep**   | Быстрый поиск по тексту, альтернатива `grep`                              |
| **sd**        | Быстрый и удобный `sed`, с синтаксисом от Rust                            |
| **fastfetch** | Быстрый вывод системной информации (аналог `neofetch`, но быстрее)        |
| **cpufetch**  | ASCII-арт с информацией о процессоре                                      |
| **procs**     | Современная замена `ps` с удобным выводом                                 |
| **du-dust**   | Альтернатива `du`, показывает сколько места занимают папки                |
| **htop**      | Интерактивный мониторинг процессов                                        |
| **glances**   | Мониторинг системы в режиме реального времени                             |
| **xh**        | Удобный CLI-клиент для HTTP-запросов (альтернатива `curl`, `httpie`)      |
| **jq**        | Парсинг и обработка JSON из терминала                                     |

### 🦀 Установка Rust (Cargo)

!!! info "Отсутствие утилит в основном репозитории Debian"

    Cargo нам нужен на тот случай, если нужных утилит не будет в основных репозитоях.

    Например, до недавнего времени там не было утилиты `eza`.

Многие современные CLI-инструменты пишутся на Rust и устанавливаются через `cargo`.
Установим Rust:

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

После установки перезапустите терминал или выполните:

```bash
source "$HOME/.cargo/env"
```

---

## ⚙️ Настройка драйверов

Корректная работа системы невозможна без установленных и настроенных драйверов. Этот шаг включает:

- **Обновление ядра** (XanMod) для лучшей производительности и совместимости;
- Установку драйверов **NVIDIA**;
- Настройку **аудио**;
- Подключение **Bluetooth**;
- Настройку **принтеров**.

### 🧠 Установка и настройка ядра XanMod

**XanMod** — это оптимизированное ядро Linux с улучшенной производительностью и низкими задержками, особенно полезно для разработки и работы в графической среде.
Подробнее про ядро можно прочитать [здесь](https://xanmod.org/).

```bash
wget -qO - https://dl.xanmod.org/archive.key | sudo gpg --dearmor -vo /etc/apt/keyrings/xanmod-archive-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/xanmod-archive-keyring.gpg] http://deb.xanmod.org releases main' | sudo tee /etc/apt/sources.list.d/xanmod-release.list
wget https://dl.xanmod.org/check_x86-64_psabi.sh
sudo chmod +x ./check_x86-64_psabi
sudo nala update && sudo nala install -y linux-xanmod-x64v3
rm -f check_x86-64_psabi
sudo shutdown -r now
```

### 🎮 Установка драйверов Nvidia

!!! warning "Про драйвера"

    У меня в компьютере стоят драйвера от **Nvidia**, однако у вас может быть хоть **Intel**, хоть **AMD**.

    Для информации по поводу ваших драйверов, обращайтесь в интернет.

```bash
sudo nala install -y nvidia-driver firmware-misc-nonfree
```

### 🔊 Настройка звука (PipeWire + WirePlumber)

**PipeWire** — современная альтернатива PulseAudio и JACK, работающая "из коробки" и подходящая для большинства систем.

```bash
sudo nala install -y pipewire
sudo nala install -y pipewire-audio-client-libraries
```

И не забываем включить демон для системы инициализации:

```bash
systemctl --user enable --now wireplumber
```

### 📶 Настройка Bluetooth

```bash
sudo nala install bluetooth bluez bluez-tools bluez-firmware bluez-cups
sudo systemctl enable bluetooth
sudo systemctl start bluetooth
```

### 🖨️ Поддержка и настройка принтеров

Для управления принтерами в Linux чаще всего используется система **CUPS (Common UNIX Printing System)**.
Она предоставляет удобный веб-интерфейс и поддержку множества моделей.

Установка CUPS и интерфейса управления:

```bash
sudo nala install cups printer-driver-all system-config-printer
sudo systemctl enable cups
sudo systemctl start cups
```

Теперь открыв [панель управления](http://localhost:631), ты можешь:

- Добавить USB или сетевой принтер;
- Настроить драйвер и параметры печати;
- Проверить очередь заданий.

!!! info "Принтеры от HP"

    Если используется принтер от HP, рекомендуется установить: `hplip-gui`.

    ```bash
    sudo nala install -y hplip-gui
    ```

### 🔄  Перезапуск системы

После установка всех драйверов рекомендуем перезагрузиться:

```bash
sudo shutdown -r now
```

---

## 🖥️ Установка и настройка рабочего окружения

### 🖥️ Установка Xorg (графический сервер)

```bash
sudo nala install -y xserver-xorg-core xserver-xorg xinit xclip dbus-x11 lxpolkit
```

### 🪟 Установка i3-окружения и сопутствующих программ

```bash
sudo nala install -y alacritty i3 sxhkd dunst picom polybar rofi nitrogen flameshot
```

| Название      | Описание                                                          |
| ------------- | ----------------------------------------------------------------- |
| **alacritty** | Быстрый и современный терминал с аппаратным ускорением            |
| **i3**        | Тайловый оконный менеджер для лёгкого управления окнами           |
| **sxhkd**     | Демон для настройки горячих клавиш                                |
| **dunst**     | Лёгкий и настраиваемый демон уведомлений                          |
| **picom**     | Композитор для эффектов прозрачности и теней                      |
| **polybar**   | Кастомизируемая панель статуса для i3 и других оконных менеджеров |
| **rofi**      | Лаунчер приложений и переключатель окон                           |
| **nitrogen**  | Утилита для настройки обоев рабочего стола                        |
| **flameshot** | Утилита для создания скриншотов                                   |

### 🔐 Настройка автоматического входа и запуска i3

Сначала установите `getty` с автоавторизацией:

```bash
sudo mkdir -p /etc/systemd/system/getty@tty1.service.d
sudo tee /etc/systemd/system/getty@tty1.service.d/override.conf <<EOF
[Service]
ExecStart=
ExecStart=-/sbin/agetty --autologin your_username --noclear %I \$TERM
EOF
```

Затем настройте в `.zprofile` или `.bash_profile` запуск **i3**:

```bash
if [[ -z $DISPLAY ]] && [[ $(tty) = /dev/tty1 ]]; then
  exec startx
fi
```

### 🔒 Разрешение выключения и перезагрузки без sudo

Чтобы отключить запрос пароля при shutdown/reboot, создайте файл:

```bash
sudo visudo -f /etc/sudoers.d/shutdown-nopasswd
```

Добавьте:

```sudoers
your_username ALL=(ALL) NOPASSWD: /sbin/shutdown, /sbin/reboot, /sbin/poweroff
```

---

## 🧰 Установка GUI-приложений

```bash
sudo nala install -y \
    firefox-esr \
    thunar \
    xfce4-taskmanager \
    pavucontrol \
    blueman \
    ark \
    galculator \
    mpv \
    zathura \
    gpicview \
    audacity \
    simplescreenrecorder \
    kdenlive
```

| Приложение             | Назначение                                         |
| ---------------------- | -------------------------------------------------- |
| `firefox-esr`          | Веб-браузер с долгосрочной поддержкой              |
| `thunar`               | Лёгкий файловый менеджер от XFCE                   |
| `xfce4-taskmanager`    | Лёгкий диспетчер задач                             |
| `pavucontrol`          | GUI для настройки звука через PulseAudio           |
| `blueman`              | Панель управления Bluetooth                        |
| `ark`                  | Архиватор для извлечения и создания архивов (Qt)   |
| `galculator`           | Простой калькулятор с GUI (GTK)                    |
| `mpv`                  | Универсальный медиаплеер для видео и аудио         |
| `zathura`              | Продвинутый PDF-читалка с клавиатурным управлением |
| `gpicview`             | Лёгкий просмотрщик изображений (LXDE)              |
| `audacity`             | Мощный редактор для записи и обработки звука       |
| `simplescreenrecorder` | Минималистичная программа для записи экрана        |
| `kdenlive`             | Мощный видео-редактор с простым интерфейсом        |

### 🎨 Кастомизация внешнего вида GUI-приложений

Для настройки внешнего вида приложений рекомендуется установить утилиты для управления темами и иконками как для GTK, так и для Qt.

```bash
sudo nala install -y \
    lxappearance \
    qt5ct qt6ct \
    qt-style-kvantum \
    arc-theme \
    papirus-icon-theme
```

Добавь следующие переменные окружения в `~/.bash_profile` или `~/.zprofile`:

```bash
export QT_QPA_PLATFORMTHEME=qt5ct
export QT6CT_PLATFORMTHEME=qt6ct
export QT_STYLE_OVERRIDE=kvantum
```

Далее настраивайте тему и иконки в `lxappearance` и `kvantum-manager` и перезапускаете систему.

### 🎨 Поддержка Nerd Fonts

Для корректного отображения символов в prompt и утилитах потребуется установить патченные шрифты:

```bash
wget https://github.com/ryanoasis/nerd-fonts/releases/download/v3.4.0/UbuntuMono.zip
unzip UbuntuMono.zip -d um
sudo mkdir /usr/share/fonts/UbuntuMonoNF
sudo cp um/*.ttf /usr/share/fonts/UbuntuMonoNF/
sudo fc-cache -fv
rm -rf um
```
