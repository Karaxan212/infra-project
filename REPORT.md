## Задание 1.3. Просмотр изменений

Дописал строку в README.md и выполнил `git diff`:

```
$ git diff
diff --git a/README.md b/README.md
index 92d3023..5549a0d 100644
--- a/README.md
+++ b/README.md
@@ -3,3 +3,4 @@
 Учебный инфраструктурный репозиторий курса

 <!-- Допишите сюда -->
+Учебный проект: инфраструктура с Docker и Ansible.
```

После `git add README.md`:

```
$ git diff
(пусто)

$ git diff --staged
diff --git a/README.md b/README.md
index 92d3023..5549a0d 100644
+Учебный проект: инфраструктура с Docker и Ansible.
```

Второй `git diff` пустой, потому что он сравнивает рабочую папку с индексом. После `git add` изменение уже лежит в индексе, и различий между ними нет. А `git diff --staged` сравнивает индекс с последним коммитом, поэтому показывает изменение.

## Задание 2.1. История коммитов

```
$ git log --oneline
caeff96 (HEAD -> main, origin/main) docs: describe project in README
f595306 feat(ansible): add playbook, inventory and config
567d648 feat(docker): add Dockerfile and index.html
913f46a docs: add README and .env.example
4ef80b8 chore: add .gitignore

$ git log --oneline --graph --all
* caeff96 (HEAD -> main, origin/main) docs: describe project in README
* f595306 feat(ansible): add playbook, inventory and config
* 567d648 feat(docker): add Dockerfile and index.html
* 913f46a docs: add README and .env.example
* 4ef80b8 chore: add .gitignore

$ git show HEAD~1
commit f595306e63f12c77106af158599f9ecae3e5556c
Author: Nurzhanov Chyngyz <nurzhanov_ch@iuca.kg>
Date:   Mon Oct 26 09:49:31 2026 +0000

    feat(ansible): add playbook, inventory and config

diff --git a/ansible/ansible.cfg b/ansible/ansible.cfg
new file mode 100644
+[defaults]
+inventory = inventory.ini
+host_key_checking = False
...
```

**HEAD~1** это коммит `f595306 feat(ansible): add playbook, inventory and config`. `HEAD` указывает на последний коммит (`caeff96`, про README), а `HEAD~1` на один шаг раньше.


## Задание 1.2. Файлы проекта

Создал `.env` (`DB_PASSWORD=secret123`) и `app.log`. Git их не показывает среди untracked, они значатся как игнорируемые:

```
$ git status --ignored
On branch main
Your branch is up to date with 'origin/main'.

Untracked files:
        REPORT.md

Ignored files:
        .env
        app.log
```

Файлы проекта закоммичены тремя отдельными коммитами:

```
567d648 feat(docker): add Dockerfile and index.html
913f46a docs: add README and .env.example
4ef80b8 chore: add .gitignore
f595306 feat(ansible): add playbook, inventory and config
```

Порядок в истории: `.gitignore`, затем README + .env.example, затем docker/, затем ansible/. Изменения отправлены на сервер через `git push`.

## Задание 2.2. Ветка и слияние

```
$ git switch -c feature/healthcheck
$ echo 'HEALTHCHECK CMD wget -qO- http://localhost/ || exit 1' >> docker/Dockerfile
$ git commit -am "feat(docker): add healthcheck"
$ git switch main
$ git merge feature/healthcheck
Updating caeff96..064c60e
Fast-forward
 docker/Dockerfile | 1 +
 1 file changed, 1 insertion(+)
$ git branch -d feature/healthcheck
Deleted branch feature/healthcheck (was 064c60e).
$ git push
   caeff96..064c60e  main -> main
$ cat docker/Dockerfile
FROM nginx:1.27-alpine
COPY index.html /usr/share/nginx/html/
HEALTHCHECK CMD wget -qO- http://localhost/ || exit 1
```

Слияние прошло в режиме Fast-forward: пока я работал в ветке, в `main` новых коммитов не появилось, поэтому Git просто сдвинул указатель `main` вперёд. Ветка `feature/healthcheck` удалена, строка HEALTHCHECK есть в Dockerfile в `main`.

## Задание 2.3. Конфликт слияния

В ветке `fix/app-port` изменил `app_port` на 9090, в `main` на 8000, затем выполнил `git merge fix/app-port`. Git сообщил о конфликте. Файл `ansible/group_vars/all.yml` с маркерами:

```
---
<<<<<<< HEAD
app_port: 8000
=======
app_port: 9090
>>>>>>> fix/app-port
app_user: deploy
```

Выбрал значение 8000 и удалил маркеры. Итоговый файл:

```
---
app_port: 8000
app_user: deploy
```

Завершил слияние: `git add ansible/group_vars/all.yml`, затем `git commit`. Merge-коммит виден в графе:

```
$ git log --oneline --graph --all
*   7b43d1c (HEAD -> main, origin/main) Merge branch 'fix/app-port'
|\
| * bc6b1dd fix(ansible): set app_port to 9090
* | 6a3a4a6 fix(ansible): set app_port to 8000
|/
* 064c60e feat(docker): add healthcheck
* caeff96 docs: describe project in README
* f595306 feat(ansible): add playbook, inventory and config
* 567d648 feat(docker): add Dockerfile and index.html
* 913f46a docs: add README and .env.example
* 4ef80b8 chore: add .gitignore
```

## Задание 3.1. Получение изменений с сервера

Отредактировал README.md в веб-интерфейсе GitHub и сохранил коммитом в `main`. Затем локально:

```
$ git pull
(вставьте вывод git pull)

$ git log --oneline
(вставьте вывод git log --oneline, где виден коммит из веб-интерфейса)
```

Коммит из веб-интерфейса появился локально после `git pull`.

## Задание 3.2. Merge Request

Создал ветку `feature/ansible-curl`, добавил в `ansible/playbook.yml` задачу установки curl, закоммитил в формате Conventional Commits:

```
$ git commit -am "feat(ansible): add curl installation task"
$ git push -u origin feature/ansible-curl
```

Открыл Pull Request в `main` с описанием: добавлена задача `Install curl` в playbook, curl нужен на серверах для проверки доступности сервисов. Слил его через веб-интерфейс и подтянул результат:

```
$ git switch main
$ git pull
Updating e9bd306..57a6646
Fast-forward
 ansible/playbook.yml | 5 +++++
 1 file changed, 5 insertions(+)

$ grep -n "curl" ansible/playbook.yml
23:    - name: Install curl
25:        name: curl
```

## Задание 3.3. Уборка веток

```
$ git branch -d feature/ansible-curl
$ git push origin --delete feature/ansible-curl
$ git fetch --prune
$ git branch -a
(вставьте вывод git branch -a)
```

Ветки `feature/ansible-curl` больше нет ни локально, ни на сервере.

## Задание 1.1. Настройка и первый коммит

```
$ git config --global user.name "Nurzhanov Chyngyz"
$ git config --global user.email "nurzhanov_ch@iuca.kg"
$ git config --global init.defaultBranch main
$ git clone git@github.com:Karaxan212/infra-project.git
Cloning into 'infra-project'...
warning: You appear to have cloned an empty repository.
$ printf '.env\n*.log\n*.retry\n' > .gitignore
$ git add .gitignore
$ git commit -m "chore: add .gitignore"
$ git push -u origin main
```

Настроил имя, почту и ветку по умолчанию, склонировал пустой репозиторий по SSH. Первым коммитом добавил только файл `.gitignore` со строками `.env`, `*.log`, `*.retry`.
