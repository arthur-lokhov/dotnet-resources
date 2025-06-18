---
title: Настройка рабочего окружения для .NET
hide:
    - navigation
---
# 🧰 Настройка рабочего окружения для .NET

Эта страница поможет вам подготовить удобное, современное и эффективное окружение для разработки .NET-приложений в Linux-среде.

Здесь рассматриваются:

- терминал и инструменты командной строки;
- базовые утилиты разработчика;
- установка Docker, Kubernetes (Minikube) и Helm;
- настройка отображения и удобства работы.

---

## 🐧 Выбор Linux-дистрибутива

Выбор дистрибутива напрямую влияет на удобство работы, доступность пакетов, стабильность и документацию.
Нет "лучшего" дистрибутива для всех, но есть — подходящий **для ваших целей и опыта**.

### 🧭 На что обратить внимание:

| Фактор                            | Почему это важно                                              |
| --------------------------------- | ------------------------------------------------------------- |
| **Стабильность пакетов**          | Для DevOps-инфраструктуры важно, чтобы всё было предсказуемо. |
| **Поддержка Docker и Kubernetes** | Не во всех дистрибутивах эти пакеты из коробки.               |
| **Скорость обновлений**           | Хотите bleeding edge? Или стабильность с минимумом сюрпризов? |
| **Сообщество и документация**     | Важно при поиске решений и примеров в интернете.              |
| **Управление пакетами**           | `apt`, `dnf`, `pacman` — разные подходы, разные привычки.     |
| **Системные особенности**         | Например, использование systemd, AppArmor, SELinux и пр.      |

### 🛠️ Популярные варианты

| Дистрибутив  | Подходит для                         | Особенности                                        |
| ------------ | ------------------------------------ | -------------------------------------------------- |
| **Ubuntu**   | Новичков и инфраструктуры            | Хорошая поддержка Docker, документация             |
| **Debian**   | Стабильной среды разработки и DevOps | Минимум лишнего, предсказуемость, но пакеты старее |
| **Fedora**   | Тех, кто хочет быть на острие        | Быстрые обновления, SELinux по умолчанию           |
| **openSUSE** | Корпоративной сборки/DevOps          | YaST, Snapper, разделение Tumbleweed/Leap          |
| **Arch**     | Продвинутых пользователей            | Максимальный контроль и новизна пакетов            |

!!! info "Автор использует: Debian"

    Причины:

    - Стабильная и минималистичная основа;
    - Отлично подходит под инфраструктуру и серверную часть;
    - Прост в автоматизации и настройке;
    - Работает "как надо" — без избыточных решений.

    Настройку Debian от автора можно найти [здесь](./extra/debian.md).

---

## 🔐 Настройка Git и SSH

Правильная настройка Git и SSH экономит часы времени и избавляет от боли при работе с несколькими репозиториями, ключами, CI/CD и pull requests.

### 📦 Шаг 1: Базовая настройка Git

#### Установка Git

```bash
sudo apt install git       # Debian/Ubuntu
sudo dnf install git       # Fedora
```

#### Глобальная конфигурация

```bash
git config --global user.name "Имя Фамилия"
git config --global user.email "email@example.com"

# Цветной вывод, более понятный лог, aliases
git config --global color.ui auto
git config --global core.editor "nvim"  # или "nano", "code --wait", "micro"
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.lg "log --oneline --graph --all"

# https://github.com/dandavison/delta
git config --global core.pager delta
git config --global interactive.diffFilter 'delta --color-only'
git config --global delta.navigate true
git config --global merge.conflictStyle zdiff3
```

### 🧠 Шаг 2: Поддержка нескольких аккаунтов/проектов

Если ты работаешь, например, и на **GitHub (лично)**, и на **GitLab (компания)** — нужно **разделять ключи и конфиги**.

#### Создание разных SSH-ключей

```bash
# Для GitHub
ssh-keygen -t ed25519 -C "yourname@github.com" -f ~/.ssh/id_ed25519_github

# Для GitLab (рабочий)
ssh-keygen -t ed25519 -C "you@company.com" -f ~/.ssh/id_ed25519_gitlab
```

#### Настройка `~/.ssh/config`

```config
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_github
    IdentitiesOnly yes

Host gitlab.company
    HostName gitlab.company.com
    User git
    IdentityFile ~/.ssh/id_ed25519_gitlab
    IdentitiesOnly yes
```

Теперь ты можешь использовать:

- *git@github.com:user/repo.git* — будет использовать личный ключ;
- *git@gitlab.company:team/repo.git* — будет использовать рабочий ключ.

#### Автоматический запуск `ssh-agent` и загрузка ключей в `~/.zshrc`

В `.zshrc` файл можно добавить(в моем конфиге уже есть) следующие строки:

```bash
if [ -z "$SSH_AUTH_SOCK" ]; then
    eval "$(ssh-agent -s)" > /dev/null
    ssh-add ~/.ssh/id_ed25519_github 2>/dev/null
    ssh-add ~/.ssh/id_ed25519_gitlab 2>/dev/null
fi
```

Эти строки:

- Запустят `ssh-agent`, чтобы тот управлял SSH-ключами.
- Будут поддерживать лишь один запущенный экземпляр `ssh-agent`.
- Добавят ключи через `ssh-add`, чтобы потом не надо было вводить пароль.

### 🧱  Шаг 3: Профильный .gitconfig

Создай конфигурации по проектам.

#### `~/.gitconfig` (глобально)

```ini
[includeIf "gitdir:~/Projects/Personal/"]
    path = ~/.gitconfig-personal

[includeIf "gitdir:~/Projects/Work/"]
    path = ~/.gitconfig-work
```

#### `~/.gitconfig-personal`

```ini
[user]
    name = John Dev
    email = john@devmail.com
[core]
    sshCommand = "ssh -i ~/.ssh/id_ed25519_github"
```

#### `~/.gitconfig-work`

```ini
[user]
    name = John Work
    email = john@company.com
[core]
    sshCommand = "ssh -i ~/.ssh/id_ed25519_gitlab"
```

---

## 🟢 Установка Node.js

Установка будет производиться через **Node Version Manager**.
`nvm` позволяет удобно управлять несколькими версиями Node.js, устанавливать LTS или актуальные релизы, переключаться между ними для разных проектов.

Установка `nvm`:

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
```

Установка Node.js через `nvm`:

```bash
nvm install --lts
```

---

## 🟣 Установка .NET SDK

Установка будет производиться через **dotnet-install.sh**.
Это рекомендованный кросс-дистрибутивный способ от Microsoft — особенно если вы не хотите устанавливать .NET из системных репозиториев.

Перед тем, как начать, нужно скачать скрипт установки.

```bash
mkdir -p ~/.dotnet && cd ~/.dotnet
curl -L https://dot.net/v1/dotnet-install.sh -o dotnet-install.sh
chmod +x dotnet-install.sh
```

После чего мы можем установить последнюю стабильную и LTS версии:

```bash
./dotnet-install.sh --channel STS
./dotnet-install.sh --channel LTS
```

А также установим необходимые инструменты:

```bash
dotnet tool install --global dotnet-ef
```

---

## 🐳 Docker, Minikube, Helm

Установка Docker:

```bash
curl -fsSL https://get.docker.com | sh
dockerd-rootless-setuptool.sh install
```

Установка Kubernetes и Minikube:

```bash
curl -LO https://dl.k8s.io/release/`curl -LS https://dl.k8s.io/release/stable.txt`/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl

curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \
  && chmod +x minikube
sudo install minikube /usr/local/bin/
rm -rf minikube
```

Установка Helm:

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

!!! info "Подробнее о инструментах"

    Подробную информацию о данных инструментах можно прочитать в соответствующих разделах:

    1. [Docker](./containers/docker/installation.md)
    2. [Kubernetes](./containers/k8s/)
    3. [Helm](./containers/helm/)

---
