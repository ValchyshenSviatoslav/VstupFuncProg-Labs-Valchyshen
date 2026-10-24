<p align="center"><b>МОНУ НТУУ КПІ ім. Ігоря Сікорського ФПМ СПіСКС</b></p>

<p align="center">
<b>Звіт з лабораторної роботи 3</b><br/>
"Конструктивний і деструктивний підходи до роботи зі списками"<br/>
дисципліни "Вступ до функціонального програмування"
</p>

<p align="right"><b>Студент</b>: Вальчишен Святослав Русланович, КВ-34</p>
<p align="right"><b>Рік</b>: 2026</p>

## Загальне завдання

Реалізуйте алгоритм сортування чисел у списку двома способами: функціонально і імперативно.
1. Функціональний варіант реалізації має базуватись на використанні рекурсії і конструюванні нових списків щоразу, коли необхідно виконати зміну вхідного списку. Не допускається використання: псевдо-функцій, деструктивних операцій, циклів.
2. Імперативний варіант реалізації має базуватись на використанні циклів і деструктивних функцій (псевдофункцій). Оригінальний список цей варіант реалізації також не має змінювати, тому перед виконанням деструктивних змін варто застосувати функцію `copy-list`. 
Не допускається використання функцій вищого порядку. Реалізована функція не має бути функціоналом.

## Варіант 2

Алгоритм сортування обміном №1 (без оптимізацій) за незменшенням.

## Лістинг функції з використанням конструктивного підходу

```lisp
(defun bubble-pass (lst)
  "Допоміжна рекурсивна функція: робить один прохід сортування обміном (спливання найбільшого елемента)."
  (cond ((null lst) nil)
        ((null (cdr lst)) lst)
        ((> (car lst) (cadr lst))
         (cons (cadr lst) (bubble-pass (cons (car lst) (cddr lst)))))
        (t (cons (car lst) (bubble-pass (cdr lst))))))

(defun functional-bubble-sort-helper (lst counter)
  "Допоміжна функція для виконання N проходів (без оптимізації)."
  (if (null counter)
      lst
      (functional-bubble-sort-helper (bubble-pass lst) (cdr counter))))

(defun functional-bubble-sort (lst)
  "Головна функція функціонального сортування обміном за незменшенням."
  (functional-bubble-sort-helper lst lst))
```

### Тестові набори та утиліти

```lisp
(defun check-functional-sort (name input expected)
  "Утиліта для перевірки функціонального сортування."
  (format t "~:[FAILED~;passed~]... ~a~%"
          (equal (functional-bubble-sort input) expected) name))

(defun test-functional-sort ()
  (check-functional-sort "test 1 (normal)" '(4 2 3 1) '(1 2 3 4))
  (check-functional-sort "test 2 (sorted)" '(1 2 3 4) '(1 2 3 4))
  (check-functional-sort "test 3 (reversed)" '(5 4 3 2 1) '(1 2 3 4 5))
  (check-functional-sort "test 4 (duplicates)" '(3 1 2 3 1) '(1 1 2 3 3))
  (check-functional-sort "test 5 (empty)" nil nil)
  (check-functional-sort "test 6 (single)" '(42) '(42)))
```

### Тестування

```lisp
CL-USER> (test-functional-sort)
passed... test 1 (normal)
passed... test 2 (sorted)
passed... test 3 (reversed)
passed... test 4 (duplicates)
passed... test 5 (empty)
passed... test 6 (single)
NIL
```

## Лістинг функції з використанням деструктивного підходу

```lisp
(defun imperative-bubble-sort (lst)
  "Імперативне сортування обміном за незменшенням з використанням циклів та псевдофункцій."
  (let ((result (copy-list lst))) ; Копіюємо список, щоб не змінювати оригінал
    ;; Зовнішній цикл: відраховує N проходів (без оптимізації)
    (do ((i lst (cdr i)))
        ((null i) result)
      ;; Внутрішній цикл: прохід по елементах для порівняння та обміну
      (do ((curr result (cdr curr)))
          ((null (cdr curr)))
        (when (> (car curr) (cadr curr))
          ;; Деструктивний обмін значень (swap)
          (let ((temp (car curr)))
            (setf (car curr) (cadr curr))
            (setf (cadr curr) temp)))))
    result))
```

### Тестові набори та утиліти

```lisp
(defun check-imperative-sort (name input expected)
  "Утиліта для перевірки імперативного сортування."
  (let* ((input-copy (copy-list input)) ; Робимо копію для перевірки, чи не змінився оригінал
         (result (imperative-bubble-sort input)))
    (format t "~:[FAILED~;passed~]... ~a (original intact: ~a)~%"
            (equal result expected)
            name
            (equal input input-copy))))

(defun test-imperative-sort ()
  (check-imperative-sort "test 1 (normal)" '(4 2 3 1) '(1 2 3 4))
  (check-imperative-sort "test 2 (sorted)" '(1 2 3 4) '(1 2 3 4))
  (check-imperative-sort "test 3 (reversed)" '(5 4 3 2 1) '(1 2 3 4 5))
  (check-imperative-sort "test 4 (duplicates)" '(3 1 2 3 1) '(1 1 2 3 3))
  (check-imperative-sort "test 5 (empty)" nil nil)
  (check-imperative-sort "test 6 (single)" '(42) '(42)))
```

### Тестування

```lisp
CL-USER> (test-imperative-sort)
passed... test 1 (normal) (original intact: T)
passed... test 2 (sorted) (original intact: T)
passed... test 3 (reversed) (original intact: T)
passed... test 4 (duplicates) (original intact: T)
passed... test 5 (empty) (original intact: T)
passed... test 6 (single) (original intact: T)
NIL
```