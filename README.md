# devops-netology  

---
## Домашнее задание к занятию «2.4. Инструменты Git»
### 1. Найдите полный хеш и комментарий коммита, хеш которого начинается на aefea.
        $ git rev-parse aefea
        aefead2207ef7e2aa5dc81a34aedf0cad4c32545

### 2. Какому тегу соответствует коммит 85024d3?
        $ git show -s --format=%s 85024d3
        v0.12.23

### 3. Сколько родителей у коммита b8d720? Напишите их хеши.
        $ git show --format=%P b8d720
        56cd7859e05c36c06b56d013b55a252d0bb7e158 9ea88f22fc6269854151c571162c5bcf958bee2b

### 4. Перечислите хеши и комментарии всех коммитов которые были сделаны между тегами v0.12.23 и v0.12.24.
        $ git log --oneline v0.12.23..v0.12.24 --skip=1
        b14b74c49 [Website] vmc provider links
        3f235065b Update CHANGELOG.md
        6ae64e247 registry: Fix panic when server is unreachable
        5c619ca1b website: Remove links to the getting started guide's old location
        06275647e Update CHANGELOG.md
        d5f9411f5 command: Fix bug when using terraform login on Windows
        4b6d06cc5 Update CHANGELOG.md
        dd01a3507 Update CHANGELOG.md
        225466bc3 Cleanup after v0.12.23 release

### 5. Найдите коммит в котором была создана функция func providerSource, ее определение в коде выглядит так func providerSource(...) (вместо троеточего перечислены аргументы).
        $ git log --oneline --pickaxe-regex -S 'func providerSource\(.*\)'
        8c928e835 main: Consult local directories as potential mirrors of providers

### 6. Найдите все коммиты в которых была изменена функция globalPluginDirs.
        $ git grep -p 'globalPluginDirs'
        commands.go=func initCommands(
        commands.go:            GlobalPluginDirs: globalPluginDirs(),
        commands.go=func credentialsSource(config *cliconfig.Config) (auth.CredentialsSource, error) {
        commands.go:    helperPlugins := pluginDiscovery.FindPlugins("credentials", globalPluginDirs())
        plugins.go=import (
        plugins.go:func globalPluginDirs() []string {
        $ git log --oneline --no-patch -L :globalPluginDirs:plugins.go
        78b122055 Remove config.go and update things using its aliases
        52dbf9483 keep .terraform.d/plugins for discovery
        41ab0aef7 Add missing OS_ARCH dir to global plugin paths
        66ebff90c move some more plugin search path logic to command
        8364383c3 Push plugin discovery down into command package

### 7. Кто автор функции synchronizedWriters?
        $ git log --pretty=format:"%h %ad %s" -G 'synchronizedWriters'
        bdfea50cc Mon Nov 30 18:02:04 2020 -0500 remove unused
        fd4f7eb0b Wed Oct 21 13:06:23 2020 -0400 remove prefixed io
        5ac311e2a Wed May 3 16:25:41 2017 -0700 main: synchronize writes to VT100-faker on Windows
        $ git checkout 5ac311e2a
        $ git grep -n -p synchronizedWriters
        main.go=234=func copyOutput(r io.Reader, doneCh chan<- struct{}) {
        main.go:267:            wrapped := synchronizedWriters(stdout, stderr)
        synchronized_writers.go=8=type synchronizedWriter struct {
        synchronized_writers.go:13:// synchronizedWriters takes a set of writers and returns wrappers that ensure
        synchronized_writers.go:15:func synchronizedWriters(targets ...io.Writer) []io.Writer {
        $ git blame --line-porcelain -L 15,15 synchronized_writers.go
        5ac311e2a91e381e2f52234668b49ba670aa0fe5 15 15 1
        author Martin Atkins
        author-mail <mart@degeneration.co.uk>
        author-time 1493853941
        author-tz -0700
        committer Martin Atkins
        committer-mail <mart@degeneration.co.uk>
        committer-time 1493937411
        committer-tz -0700
        summary main: synchronize writes to VT100-faker on Windows
        filename synchronized_writers.go
        func synchronizedWriters(targets ...io.Writer) []io.Writer {
