#devops-netology  

---
##Домашнее задание к занятию «2.3. Ветвления в Git»
### 1. Подготовка
Создан каталог `branching` внутри которого созданы файлы `merge.sh` и `rebase.sh`, содержащие bash скрипт.
Создан коммит с описанием `prepare for merge and rebase` и отправлен в ветку main удаленного репозитория:

        $ git add .
        $ git commit -m "prepare for merge and rebase"
### 2. Подготовка файла merge.sh
Создана ветка git-merge: 

        $ git checkout -b git-merge 
Внесены требуемые правки в файл `merge.sh` и отправлены в репозиторий:

        $ git add .
        $ git commit -m "merge: @ instead *"
        $ git push -u origin git-merge
Добавлены новые изменения в файл `merge.sh`, закоммичены и отправлены в репозиторий:

        $ git add .
        $ git commit -m "merge: use shift"
        $ git push -u origin git-mereg
### 3. Изменения в ветке main
Следующим шагом в ветке main произведены изменения в файле rebase.sh и отправлены в репозиторий:

        $ git switch main
        $ nano branching/rebase.sh
        $ git add .
        $ git commit -m "edit rebase.sh in main"
        $ git push -u origin main
### 4.Подготовка файла rebase.sh
Найден хэш коммита prepare for merge and rebase, о найденного коммита создана новая ветка git-rebase:   

        $ git log --oneline | grep 'prepare for merge and rebase'
        $ git checkout f311cd4
        $ git switch -c git-rebase
В файл rebase.sh внесены изменения, закоммичены и отправлены в репозиторий в ветку git-rebase:

        $ git add .
        $ git commit -m "git-rebase 1"
        $ git push -u origin git-rebase
Следующее изменение в файле rebase.sh, с сообщением коммита `git-rebase 2` отправлено в репозиторий:
        
        $ git add.
        $ git commit -m "git-rebase 2"
        $ git push -u origin git-rebase
### 5. Промежуточный итог
![Созданы обе ветки](https://c.radikal.ru/c03/2111/d0/ff30a0dd45e5.png)
> Скриншот графического отображения состояния веток репозитория https://github.com/martynov-andrey/devops-netology/network
### 6. Merge
Произведено слияние ветки git-merge, и передано в репозиторий: 

        $ git checkout main
        $ git merge git-merge
        $ git push
![Merge git-gerge](https://d.radikal.ru/d08/2111/ce/0bd7b1ec624d.png)
### 7. Rebase
Выполнена операция перебазирования ветки git-rebase в ветку main: 
        
        $ git switch git-rebase
        $ git rebase -i main
В открывшемся диалоговом окне произведена замена в нижней строчке pickup на fixup:

        pick 0f15197 git-rebase 1
        fixup 93d11ee git-rebase 2
В файле rebase.sh разрешен конфликт, продолжен ребейз:

        $ git add branching/rebase.sh
        $ git rebase --continue
Повторно разрешен следующий конфликт в файле rebase.sh и завершен ребейз ветки:

        $ git add branching/rebase.sh
        $ git rebase --continue
Сообщение об успешном завершении операции:   

`[detached HEAD 2ecb259] Merge branch 'git-merge' into main Date: Thu Nov 4 20:37:51 2021 +0300 Successfully rebased and updated refs/heads/git-rebase.`  

Сохранить изменения в ветке в репозиторий возможно с флагом force: 
        
        $ git push -u origin git-rebase -f

![Ветка git-rebase](https://a.radikal.ru/a20/2111/da/e40dc25afa44.png)
Мердж ветки git-merge в main:

        $ git checkout main
        $ git merge git-rebase
Происходит с сообщением:  

`Merge made by the 'recursive' strategy.`  
`branching/rebase.sh | 2 +-`  
`1 file changed, 1 insertion(+), 1 deletion(-)`  

Собщение указывает на слияние простым трехсторонним методом, что расходится с описанием в задании.

![Конечный результат](https://a.radikal.ru/a19/2111/fd/de92f94e0f66.png)