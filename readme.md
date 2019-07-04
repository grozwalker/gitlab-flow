### Git flow

#### Работа с ветками

* Главная ветка `master` (QA). `roduction` (Prod) - ветка для прода. Эти ветки закрыты для записи напрямую.

* Каждая задача (фича\багфикс и т.д.) в новой ветке. При выполнении задач ветвиться от ветки `master`: `git checkout -b new-branch`

* Чтобы получить данные о новых ветках на удаленном репозитории, выполняем `git fetch`, но это не обновит текущую локальную ветку 

* Если мы хотим обновить текущую локальную ветку из удаленного репозитория, делаем `git pull --no-edit origin $(git rev-parse --abbrev-ref HEAD) --rebase --ff-only`. Это предотвратит лишний merge-коммит и будет держать репозитрой в чистоте

* Когда задача доделана, нам надо показать код тестироващика и пройти кросс-ревью (код-ревью). Для этого мы пушим изменения и в интерфейсе гитлаба создаем Merge Request (MR), назначаем людей для ревью.

* Гитлаб предоставляет технологию review-app: на каждый MR создается свой поддомен, где можно смотреть все изменения. Поддомен вида: mr_name.itech-group.ru

* Во время прохождения ревью и тестирования, происходят новые пуш-коммиты, поэтому после того, как вам будет поставлен аппрув (количество аппрувов при кросс-ревью будет обговариваться отдельно, в зависимости от количества участников) нужно будет засквошить все ненужные коммиты и оставить только те, которые нужны (исходить из здравого смысла)

* Мерджим нашу ветку в мастер, старая ветка удаляется (можно и оставлять) и запускается пайплайн для тестирования ветки мастер и обновления qa.

* Когда приходит время выпуска нового релиза - мерджим наши изменения в ветку `production` и в ручном режиме запускаем тот же процесс тестов и деплоя кода на продовский сервер

#### Тестирование ветки с фичами

В гитлабе реалзиована фича с пайплайнами, которые позволяют запускать автоматические тесты на каждый коммит. Для этого надо создать файл gitlab-ci.yml, где прописать основные команды для разворачивания поддоменов для МР (review app), тестирования и деплоя веток.

В целом процесс выглядит вот так: 
![flow with review app](https://docs.gitlab.com/ee/ci/review_apps/img/continuous-delivery-review-apps.svg)

При каждом коммите должна запускаться автопроверка следующих вещей:

- Linter (PHP Code Sniffer) - чтобы во-первых было следование стандартам, во-вторых, единый стилоь на фирме.

- Unit тесты


#### Жизненный цикл разработки с точки зрения гита
```git
> git checkout master                   # переходим на главную ветку
> git status                            # убеждаемся, что нет никаких изменений
> git pull --no-edit origin \
  $(git rev-parse --abbrev-ref HEAD) \
  --rebase --ff-only                    # забираем последние изменения
> git checkout -b new-branch            # создаем и переходим на новую ветку
> git add file-name                     # добавляем файлы к коммиту
> git commit                            # создаем коммит
> git push -u origin $(git rev-parse --abbrev-ref HEAD) # пушим изменения

```

P.S.
Для удобства можно создать алиасы команд для bash
`nano ~/.bashrc`

```git
    alias push='git push -u origin $(git rev-parse --abbrev-ref HEAD) $@'
    
    alias rebase='git pull --no-edit origin $(git rev-parse --abbrev-ref HEAD) --rebase --ff-only'

```


#### Коммит
Когда задача закончена или мы по каким-то причинам решаем сделать коммит.
 
`> git status` - смотрим изменения в рабочем дереве (обязательно проверяем, что в коммит не попадут папки вроде .idea, node_modules и прочее)

`> git add path/to/file` или `git add -A` (добавляем сразу все файлы, но надо внимательно посмотреть, чтобы ничего лишнего не попало)

`> git commit` - в окне редактора пишем осмысленное описание того, что сделали (не bugfix, а что было сделано).
Желательно оформлять коммит таким образом:
```
Commit subject          - Заголовок коммита
                        - Пустая строка
Commit body             - Тело коммита
                        - Пустая строка
Fixes #123              - Задача из jira
```
Заголовок коммита:
 - начинается с большой буквы 
 - длина до 50 символов в конце точку не ставить
 - использование повелительного наклонения глагола (сделай, слей, исправь). Никаких fixed, changed и т.д.
 
 
Тело коммита:
 - Длина строки до 72 символа
 - Раскрывает что сделано и зачем, а не как это сделано 
 
 
Задача из jira (интеграция jira-gitlab):
 - Номер задачи из джиры

### Сквош коммитов
Если вдруг вы в своей ЛОКАЛЬНОЙ ветке выполнили `git log` и совершенно случайно обнаружили список вот таких коммитов:
```git
bcdca61 fix 1
4643a5f fix header
e0ca8b9 update branch
fg8oo56 fixed header
```

То это не повод переносить все это в общий репозиторий, для этого нам надо обьеденить коммиты (засквошить)
```git
> Закоммитить или спрятать все изменения, если есть
> git log #Чтобы понимать какое количество коммитов надо надо обьеденить. В нашем случае 4
> git rebase -i HEAD~4 
```
Откроется диалоговое окно, где будет список из четырех коммитов и напротив каждого из них будет написано pick. Коммиты идут в порядке возрастания времени создания. Самый нижний - самый свежий.
```git
pick bcdca61 fix 1
pick 4643a5f fix header
pick e0ca8b9 update branch
pick fg8oo56 fixed header
```

Напротив тех коммитов, которые надо объеденить пишем squash

```git
pick bcdca61 fix 1
squash 4643a5f fix header
squash e0ca8b9 update branch
squash fg8oo56 fixed header
```
Закрываем окно редактора и в следующем окне вводим название общего коммита.

ВНИМАНИЕ! Это перезаписывает историю гита, можно выполнять только в ветке, где вы работаете один, ни в коем случае этого нельзя делать в ветке `production` или `master`
