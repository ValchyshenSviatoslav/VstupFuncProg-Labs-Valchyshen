<p align="center"><b>МОНУ НТУУ КПІ ім. Ігоря Сікорського ФПМ СПіСКС</b></p>

<p align="center">
<b>Звіт з лабораторної роботи 4</b><br/>
"Функції вищого порядку та замикання"<br/>
дисципліни "Вступ до функціонального програмування"
</p>

<p align="right"><b>Студент(-ка)</b>: Вальчишен Святослав Русланович, КВ-34</p>
<p align="right"><b>Рік</b>: 2026</p>

## Загальне завдання

Завдання складається з двох частин:
1. Переписати функціональну реалізацію алгоритму сортування з лабораторної роботи 3 з такими змінами:
   - використати функції вищого порядку для роботи з послідовностями;
   - додати до інтерфейсу функції два ключових параметра: `key` та `test`, що працюють аналогічно до стандартних. При цьому `key` має виконатись мінімальну кількість разів.
2. Реалізувати функцію, що створює замикання, яке працює згідно із завданням за варіантом. Використання псевдофункцій не забороняється, але має бути зведене до мінімуму.

## Варіант першої частини 2

Алгоритм сортування обміном №1 (без оптимізацій) за незменшенням.

## Лістинг реалізації першої частини завдання

```lisp
(defun bubble-pass-decorated (lst test)
  "Один прохід сортування для задекорованого списку (порівнюються попередньо обчислені ключі)."
  (cond ((null lst) nil)
        ((null (cdr lst)) lst)
        ;; Якщо другий ключ "кращий" за перший згідно test (порушення порядку), міняємо їх місцями
        ((funcall test (caadr lst) (caar lst))
         (cons (cadr lst) (bubble-pass-decorated (cons (car lst) (cddr lst)) test)))
        (t (cons (car lst) (bubble-pass-decorated (cdr lst) test)))))

(defun functional-bubble-sort-decorated (lst test counter)
  "Рекурсивно виконує N проходів сортування для задекорованого списку."
  (if (null counter)
      lst
      (functional-bubble-sort-decorated (bubble-pass-decorated lst test) test (cdr counter))))

(defun functional-bubble-sort-hof (lst &key (key #'identity) (test #'<))
  "Головна функція сортування. Використовує HOF (mapcar) та патерн Decorate-Sort-Undecorate 
   для того, щоб викликати функцію :key мінімальну кількість разів (по 1 разу на елемент)."
  (let* (;; Decorate: створюємо список пар виду (ключ . елемент)
         (decorated (mapcar (lambda (x) (cons (funcall key x) x)) lst))
         ;; Sort: сортуємо задекорований список, використовуючи тест для ключів
         (sorted-decorated (functional-bubble-sort-decorated decorated test decorated)))
    ;; Undecorate: розпаковуємо відсортовані елементи
    (mapcar #'cdr sorted-decorated)))
```

### Тестові набори та утиліти першої частини

```lisp
(defun check-functional-sort-hof (name input expected &key (key #'identity) (test #'<))
  "Утиліта для перевірки сортування з ключовими параметрами."
  (format t "~:[FAILED~;passed~]... ~a~%"
          (equal (functional-bubble-sort-hof input :key key :test test) expected)
          name))

(defun test-functional-sort-hof ()
  (check-functional-sort-hof "test 1 (default <)" '(4 2 3 1) '(1 2 3 4))
  (check-functional-sort-hof "test 2 (custom test >)" '(4 2 3 1) '(4 3 2 1) :test #'>)
  (check-functional-sort-hof "test 3 (custom key abs)" '(-4 2 -3 1) '(1 2 -3 -4) :key #'abs)
  (check-functional-sort-hof "test 4 (key and test on pairs)" 
                             '((a . 3) (b . 1) (c . 2)) 
                             '((b . 1) (c . 2) (a . 3)) 
                             :key #'cdr :test #'<)
  (check-functional-sort-hof "test 5 (empty list)" nil nil))
```

### Тестування першої частини

```lisp
CL-USER> (test-functional-sort-hof)
passed... test 1 (default <)
passed... test 2 (custom test >)
passed... test 3 (custom key abs)
passed... test 4 (key and test on pairs)
passed... test 5 (empty list)
NIL
```

## Варіант другої частини 2

Написати функцію `add-prev-reducer`, яка має один ключовий параметр — функцію transform. Вона має повернути функцію, яка при застосуванні в якості першого аргументу `reduce` перетворює кожен елемент списку-аргументу на точкову пару, де в комірці CAR знаходиться значення поточного елемента, а в комірці CDR — значення попереднього елемента. Якщо `transform` передана, значення елементів, що потраплять у результат, мають бути змінені згідно з цією функцією (з мінімальною кількістю викликів `transform`). Обмеження на `reduce` визначаються розробником.

## Лістинг реалізації другої частини завдання

```lisp
(defun add-prev-reducer (&key transform)
  "Повертає замикання-редуктор, яке формує пари (поточний . попередній).
   ОБМЕЖЕННЯ: функцію reduce необхідно викликати з параметрами :initial-value nil та :from-end nil."
  (let ((prev-transformed nil)) ; Локальний стан замикання
    (lambda (acc curr)
      (let ((curr-transformed (if transform (funcall transform curr) curr)))
        ;; prog1 повертає результат append, але перед цим оновлює стан prev-transformed
        (prog1
            (append acc (list (cons curr-transformed prev-transformed)))
          (setf prev-transformed curr-transformed))))))
```

### Тестові набори та утиліти другої частини

```lisp
(defun check-add-prev-reducer (name input expected &key transform)
  "Утиліта для перевірки роботи редуктора."
  (let ((reducer (if transform (add-prev-reducer :transform transform) (add-prev-reducer))))
    (format t "~:[FAILED~;passed~]... ~a~%"
            (equal (reduce reducer input :initial-value nil) expected)
            name)))

(defun test-add-prev-reducer ()
  (check-add-prev-reducer "test 1 (without transform)" 
                          '(1 2 3) 
                          '((1 . NIL) (2 . 1) (3 . 2)))
  (check-add-prev-reducer "test 2 (with transform 1+)" 
                          '(1 2 3) 
                          '((2 . NIL) (3 . 2) (4 . 3)) 
                          :transform #'1+)
  (check-add-prev-reducer "test 3 (empty list)" 
                          nil 
                          nil)
  (check-add-prev-reducer "test 4 (symbols with custom transform)" 
                          '(a b c) 
                          '((A-T . NIL) (B-T . A-T) (C-T . B-T))
                          :transform (lambda (x) (intern (concatenate 'string (symbol-name x) "-T")))))
```

### Тестування другої частини

```lisp
CL-USER> (test-add-prev-reducer)
passed... test 1 (without transform)
passed... test 2 (with transform 1+)
passed... test 3 (empty list)
passed... test 4 (symbols with custom transform)
NIL
```