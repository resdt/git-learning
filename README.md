# Git-learning
Сейчас расскажу, что вообще происходит.
MAI-GitLab нам необходим для того, чтобы показать проект, но он довольно неудобен. Поэтому я подумал, почему бы не вести разработку в удобном GitHub, а в MAI-GitLab заливать формальные коммиты и релизы?
Так почему же здесь два репозитория?
Первый репозиторий был создан просто, чтобы сохранить файлы. Из-за незнания Git я испортил ветвление. Это далеко не критично, но некрасиво. Поэтому все файлы проекта были перенесены во второй репозиторий с правильным ветвлением, а старый репозиторий можно использовать как среду для тестирования Git и GitHub. По поводу ветвления можете прочитать в [этой](https://byurrer.ru/git-workflow) статье. Наша стратегия ветвления — **Git flow** (на рисунке показан универсальный граф; в стратегии Git flow не обращайте внимания на ветки, которые находятся правее ветки main).
В GitHub есть все функции, которые нам необходимы:

- Например, доска с заданиями — она уже вынесена во вкладке `Projects`.
- Необходимо просматривать коммиты и следить за стратегией ветвления? Переходите во вкладку `Insights -> Network`, и вы увидите лаконичный график коммитов.
- Нужно что-то обсудить касаемо проекта? В GitHub есть обсуждения внутри организации по определённым категориям

![graph](https://user-images.githubusercontent.com/93484137/235647635-c4464bb3-fef6-4dc4-a233-3eeebbc02750.png)

Теперь по поводу проекта. Мы будем использовать семантическое версионирование (о нем вы можете прочитать в [этой](https://semver.org/lang/ru/) статье). Сейчас объясню, как будет происходить ветвление в нашем проекте. У нас есть основная ветка разработки — `develop`, но синхронизировать `develop` с удаленным репозиторием можно только при незначительных улучшениях или при исправлении ошибок. Если вы хотите добавить новый функционал, то необходимо создать новую ветку `<feature>`, которая обязательно должна иметь название вашего функционала (например, при добавлении новой локации ветку следует назвать "new-location" или "second-floor"). После того, как вы создали новый функционал, его необходимо протестировать. Для этого вы создаете (создаете, копируя ветку `develop`) ветку с названием `QA`, После чего вливаете ветку `<feature>` в ветку `QA`. Теперь запускаете и тестируете проект. Если тестирование оказалось неудачным, то вы просто дорабатываете ветку `<feature>`, а потом снова вливаете `<feature>` в `QA` и тестируете еще раз. Если тестирование оказалось успешным, то вы вливаете ветку `QA` в `develop`, затем удаляете ветки `QA` и `<feature>`. После завершения основной работы можно сделать pre-release. Он создается путем копирования ветки `develop` в новую ветку `pre-release-<#>`, где вместо решетки вы пишете версию pre-release. Над этой веткой еще можно провести работу, **но только по исправлению ошибок**. Теперь необходимо влить `pre-release-<#>` в `develop`. После этого ветка `pre-release-<#>` будет влита в ветку `main`, в которой находится только стабильный код и которая состоит _только_ из того, что попадает в нее путем вливания в нее `pre-release-<#>` и путем вливания в нее ветки по исправлению внезапных ошибок `hotfix-<#>`. Далее в ветке `main` создается специальный тег с версией и кратким описанием, а ветка `pre-release-<#>` удаляется. **Ветка `develop` может постоянно изменяться в процессе вашей работы, именно поэтому построена такая сложная система слияния веток.**

![image](https://user-images.githubusercontent.com/93484137/235672297-6de68d5d-3e7b-4e6e-8a7c-4cc50cc4de95.png)

### **Алгоритм организации ветвления проекта**

1. Создать ветку `main` в GitHub (создать новый пустой репозиторий проекта)
2. Заполнить паспорт проекта в файле `README.md`
3. Создать тег `v0.1.0` в ветке `main`
```
git checkout main
git pull origin main
git tag -a v0.1.0 -m "Project passport"
git push --tags
```
4. Клонировать ветку `main` в ветку `develop`
```
git checkout -b develop main
```
5. Добавить файлы в локальную папку проекта в ветке `develop`
6. Синхронизировать ветку `develop` с удаленным репозиторием
```
git add .
git commit -m "<commit>"
git push origin develop
```
7. Клонировать ветку `develop` в ветку `<feature>`
```
git checkout -b <feature> develop
```
8. Добавить файлы в локальную папку проекта в ветке `<feature>`
9. Синхронизировать ветку `<feature>` с удаленным репозиторием
```
git add .
git commit -m "<commit>"
git push origin <feature>
```
10. Клонировать ветку `develop` в ветку `QA`
```
git checkout -b QA develop
```
11. Влить ветку `<feature>` в ветку `QA`
```
git merge --no-ff <feature>
git push origin QA
```
12. Протестировать проект
13. Влить ветку `QA` в ветку `develop`
```
git checkout develop
git pull origin develop
git merge --no-ff QA
git push origin develop
```
14. Удалить ветки `QA` и `<feature>`
```
git push origin -d QA <feature>
```
15. Клонировать ветку `develop` в ветку `pre-release-<#>`
```
git checkout -b pre-release-<#> develop
git push origin pre-release-<#>
```
16. Влить ветку `pre-release-<#>` в ветку `main`
```
git checkout main
git pull
git merge --no-ff pre-release-<#>
git push
```
17. Создать тег `v0.2.0` в ветке `main`
```
git checkout main
git tag -a v0.2.0 -m "Playable concept"
git push --tags
```
18. Удалить ветку `pre-release-<#>`
```
git push origin -d pre-release-<#>
```
19. Клонировать ветку `main` в ветку `pre-production`
```
git checkout -b pre-production main
git push origin pre-production
```
20. Создать тег `v1.0.0-alpha` в ветке `pre-production`
```
git checkout pre-production
git tag -a v1.0.0-alpha -m "Tester testing"
git push --tags
```
21. Клонировать ветку `pre-production` в ветку `production`
```
git checkout -b production pre-production
git push origin production
```
22. Создать тег `v1.0.0-beta` в ветке `production`
```
git checkout production
git tag -a v1.0.0-beta -m "Public testing"
git push --tags
```
23. Клонировать ветку `production` в ветку `release`
```
git checkout -b release production
git push origin release
```
24. Создать тег `1` в ветке `release`
```
git checkout release
git tag -a 1 -m "Official release"
git push --tags
```
