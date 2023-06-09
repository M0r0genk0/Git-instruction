## Работа с конфликтами
![](/photo/%D0%B7%D0%B0%D0%B3%D1%80%D1%83%D0%B6%D0%B5%D0%BD%D0%BE.gif)
---
Системы контроля версий предназначены для управления дополнениями, вносимыми в проект множеством распределенных авторов (обычно разработчиков). Иногда один и тот же контент могут редактировать сразу несколько разработчиков. Если разработчик A попытается изменить код, который редактирует разработчик B, может произойти конфликт. Для предотвращения конфликтов разработчики работают в отдельных изолированных ветках. Основная задача команды `git merge` заключается в слиянии отдельных веток и разрешении любых конфликтующих правок.

### **Общие сведения о конфликтах слияния**

Слияние и конфликты являются неотъемлемой частью работы с Git. В других инструментах управления версиями, например SVN, работа с конфликтами может быть дорогой и времязатратной. Git позволяет выполнять слияния очень просто. В большинстве случаев Git самостоятельно решает, как автоматически интегрировать новые изменения.

Обычно конфликты возникают, когда два человека изменяют одни и те же строки в файле или один разработчик удаляет файл, который в это время изменяет другой разработчик. В таких случаях Git не может автоматически определить, какое изменение является правильным. Конфликты затрагивают только того разработчика, который выполняет слияние, остальная часть команды о конфликте не знает. Git помечает файл как конфликтующий и останавливает процесс слияния. В этом случае ответственность за разрешение конфликта несут разработчики.

### **Типы конфликтов слияния**
Конфликт во время слияния может произойти в двух отдельных точках — при запуске и во время процесса слияния. Далее рассмотрим, как разрешать каждый из этих конфликтных сценариев.

Git прерывает работу в самом начале слияния
Выполнение команды слияния прерывается в самом начале, если Git обнаруживает изменения в рабочем каталоге или разделе проиндексированных файлов текущего проекта. Git не может выполнить слияние, поскольку иначе эти ожидающие изменения будут перезаписаны новыми коммитами. Такое случается из-за конфликтов не с другими разработчиками, а с ожидающими локальными изменениями. Локальное состояние необходимо стабилизировать с помощью команд `git stash`, `git checkout`, `git commit` или `git reset`. Если команда слияния прерывается в самом начале, выдается следующее сообщение об ошибке:
```
error: Entry '<fileName>' not uptodate. Cannot merge. (Changes in working directory)
```
Git прерывает работу во время слияния
Сбой В процессе слияния говорит о наличии конфликта между текущей локальной веткой и веткой, с которой выполняется слияние. Это свидетельствует о конфликте с кодом другого разработчика. Git сделает все возможное, чтобы объединить файлы, но оставит конфликтующие участки, чтобы вы разрешили их вручную. При сбое во время выполнения слияния выдается следующее сообщение об ошибке:
```
error: Entry '<fileName>' would be overwritten by merge. Cannot merge. (Changes in staging area)
```
### **Создание конфликта слияния**
Чтобы лучше разобраться в конфликтах слияния, в следующем разделе мы смоделируем конфликт для дальнейшего изучения и разрешения. Для запуска моделируемого примера будет использоваться интерфейс Git c Unix-подобной командной строкой.
```
$ mkdir git-merge-test
$ cd git-merge-test
$ git init .
$ echo "this is some content to mess with" > merge.txt
$ git add merge.txt
$ git commit -am"we are commiting the inital content"
[main (root-commit) d48e74c] we are commiting the inital content
1 file changed, 1 insertion(+)
create mode 100644 merge.txt
```
С помощью приведенной в этом примере последовательности команд выполняются следующие действия.

Создается новый каталог с именем *git-merge-test*, выполняется переход в этот каталог и инициализация его как нового репозитория Git.
Создается новый текстовый файл *merge.txt* с некоторым содержимым.
В репозиторий добавляется файл *merge.txt* и выполняется коммит.
Теперь у нас есть новый репозиторий с одной веткой *main* и непустым файлом *merge.txt*. Далее создадим новую ветку, которая будет использоваться как конфликтующая при слиянии.
```
$ git checkout -b new_branch_to_merge_later
$ echo "totally different content to merge later" > merge.txt
$ git commit -am"edited the content of merge.txt to cause a conflict"
[new_branch_to_merge_later 6282319] edited the content of merge.txt to cause a conflict
1 file changed, 1 insertion(+), 1 deletion(-)
```
Представленная выше последовательность команд выполняет следующие действия.

  * Создает новую ветку с именем `new_branch_to_merge_later` и выполняет переход в нее.
  * Перезаписывает содержимое файла `merge.txt`.
  * Выполняет коммит нового содержимого.

В этой новой ветке `new_branch_to_merge_later` мы создали коммит, который переопределил содержимое файла `merge.txt`.
```
git checkout main
Switched to branch 'main'
echo "content to append" >> merge.txt
git commit -am"appended content to merge.txt"
[main 24fbe3c] appended content to merge.tx
1 file changed, 1 insertion(+)
```
Эта последовательность команд выполняет переключение на ветку `main`, добавляет содержимое в файл `merge.txt` и делает коммит. После этого в нашем экспериментальном репозитории находятся два новых коммита, первый — в ветке `main`, а второй — в ветке `new_branch_to_merge_later`. Теперь запустим команду `git merge new_branch_to_merge_later` и посмотрим, что из этого выйдет!
```
$ git merge new_branch_to_merge_later
Auto-merging merge.txt
CONFLICT (content): Merge conflict in merge.txt
Automatic merge failed; fix conflicts and then commit the result.
```
БАХ! 💥 Возник конфликт. Хорошо, что система Git сообщила нам об этом.

### **Выявление конфликтов слияния**
Как мы убедились на выполняемом примере, Git выводит небольшое описательное сообщение о возникновении КОНФЛИКТА. Чтобы получить более глубокое понимание проблемы, можно запустить команду `git status`.
```
$ git status
On branch main
You have unmerged paths.
(fix conflicts and run "git commit")
(use "git merge --abort" to abort the merge)

Unmerged paths:
(use "git add <file>..." to mark resolution)

both modified:   merge.txt
```
Вывод команды git status говорит о том, что из-за конфликта не удалось слить пути. Теперь файл merge.text отображается как измененный. Давайте изучим этот файл и посмотрим, что изменилось.
```
$ cat merge.txt
<<<<<<< HEAD
this is some content to mess with
content to append
=======
totally different content to merge later
>>>>>>> new_branch_to_merge_later
```
Для просмотра содержимого файла merge.txt воспользуемся командой cat. Видно, что в файле появились новые странные дополнения:
```
<<<<<<< HEAD
=======
>>>>>>> new_branch_to_merge_later
```
Эти новые строки можно рассматривать как *«разделители конфликта»*. Строка ======= является *«центром»* конфликта. Все содержимое между этим центром и строкой <<<<<<< `HEAD` находится в текущей ветке `main`, на которую ссылается указатель `HEAD`. А все содержимое между центром и строкой >>>>>>> `new_branch_to_merge_later` является содержимым ветки для слияния.

### **Разрешение конфликтов слияния с помощью командной строки**
Самый простой способ разрешить конфликт — отредактировать конфликтующий файл. Откройте файл `merge.txt` в привычном редакторе. В нашем примере просто удалим все разделители конфликта. Измененное содержимое файла `merge.txt` будет выглядеть следующим образом:
```
this is some content to mess with
content to append
totally different content to merge later
```
После редактирования файла выполните команду `git add merge.txt`, чтобы добавить новое объединенное содержимое в раздел проиндексированных файлов. Для завершения слияния создайте новый коммит, выполнив следующую команду:
```
git commit -m "merged and resolved the conflict in merge.txt"
```
Git обнаружит, что конфликт разрешен, и создаст новый коммит слияния для завершения процедуры слияния.

### **Команды Git, с помощью которых можно разрешить конфликты слияния**
#### *Общие инструменты*
`git status` - Команда *status* часто используется во время работы с Git и помогает идентифицировать конфликтующие во время слияния файлы.

`git log --merge` - При передаче аргумента *--merge* для команды git log будет создан журнал со списком конфликтов коммитов между ветками, для которых выполняется слияние.

`git diff` - Команда *diff* помогает найти различия между состояниями репозитория/файлов. Она полезна для выявления и предупреждения конфликтов слияния.

#### *Инструменты для случаев, когда Git прерывает работу в самом начале слияния*
`git checkout` - Команда *checkout* может использоваться для отмены изменений в файлах или для изменения веток.

`git reset --mixed` - Команда *reset* может использоваться для отмены изменений в рабочем каталоге или в разделе проиндексированных файлов.

#### *Инструменты для случаев, когда конфликты Git возникают во время слияния*
`git merge --abort` - При выполнении команды *git merge* с опцией *--abort* процесс слияния будет прерван, а ветка вернется к состоянию, в котором она находилась до начала слияния.

`git reset` - Команду *git reset* можно использовать для разрешения конфликтов, возникающих во время выполнения слияния, чтобы восстановить заведомо удовлетворительное состояние конфликтующих файлов.

[< К содержанию](Readme.md)