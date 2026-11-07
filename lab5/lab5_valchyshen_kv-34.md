<p align="center"><b>МОНУ НТУУ КПІ ім. Ігоря Сікорського ФПМ СПіСКС</b></p>

<p align="center">
<b>Звіт з лабораторної роботи 5</b><br/>
"Робота з базою даних"<br/>
дисципліни "Вступ до функціонального програмування"
</p>

<p align="right"><b>Студент(-ка)</b>: Вальчишен Святослав Русланович, КВ-34</p>
<p align="right"><b>Рік</b>: 2026</p>

## Загальне завдання

В роботі необхідно реалізувати утиліти для роботи з базою даних. База даних складається з кількох таблиць, представлених у вигляді CSV файлів. 
1. Визначити утиліти для створення записів з таблиць.
2. Розробити утиліту для зчитування таблиць з файлів, де числові колонки розібрані як числа.
3. Розробити функцію `select`, яка повертає лямбда-вираз для фільтрації записів.
4. Написати утиліту для запису вибірки у файл.
5. Написати функцію для конвертування записів у інший тип.
6. Написати функцію для "красивого" виводу записів таблиці (`pretty-print`).

## Варіант 2

База даних: Виробництво дронів (таблиці: 1. Виробники дронів, 2. Дрони)
Тип записів: Геш-таблиця

## Лістинг реалізації завдання

```lisp
(require 'uiop)

(defun parse-value (val-str)
  (let ((num (parse-integer val-str :junk-allowed t)))
    (if (and num (= (length (write-to-string num)) (length val-str)))
        num
        val-str)))

(defun create-record-hash (headers values)
  (let ((hash (make-hash-table :test 'eql)))
    (loop for h in headers
          for v in values
          do (setf (gethash h hash) (parse-value v)))
    hash))

(defun read-csv-to-hashes (filepath)
  (with-open-file (stream filepath :direction :input :if-does-not-exist nil)
    (when stream
      (let* ((header-line (read-line stream nil))
             (headers (mapcar (lambda (h) (intern (string-upcase h) "KEYWORD"))
                              (uiop:split-string header-line :separator ",")))
             (records nil))
        (loop for line = (read-line stream nil)
              while line
              do (let ((values (uiop:split-string line :separator ",")))
                   (push (create-record-hash headers values) records)))
        (nreverse records)))))

(defun select (filepath)
  (let ((all-records (read-csv-to-hashes filepath)))
    (lambda (&rest filters)
      (if (null filters)
          all-records
          (remove-if-not 
           (lambda (record)
             (loop for (key val) on filters by #'cddr
                   always (equal (gethash key record) val)))
           all-records)))))

(defun write-hashes-to-csv (records filepath)
  (when records
    (with-open-file (stream filepath :direction :output :if-exists :supersede)
      (let ((headers (loop for k being the hash-keys of (first records) collect k)))
        (format stream "~{~a~^,~}~%" (mapcar #'symbol-name headers))
        (loop for rec in records do
          (let ((vals (mapcar (lambda (h) (gethash h rec)) headers)))
            (format stream "~{~a~^,~}~%" vals)))))))

(defun hash-to-alist (hash)
  (let ((alist nil))
    (maphash (lambda (k v) (push (cons k v) alist)) hash)
    (nreverse alist)))

(defun convert-records (records)
  (mapcar #'hash-to-alist records))

(defun pretty-print-hashes (records)
  (if (null records)
      (format t "Empty~%")
      (loop for rec in records
            for i from 1
            do (progn
                 (format t "--- ~a ---~%" i)
                 (maphash (lambda (k v) (format t "  ~a: ~a~%" k v)) rec))))
  (values))
```

### Вміст тестових файлів

**Файл `manufacturers.csv`**
```csv
id,name,country
1,DJI,China
2,Skydio,USA
3,Parrot,France
```

**Файл `drones.csv`**
```csv
id,model,manufacturer_id,weight
101,Mavic 3,1,895
102,Skydio 2+,2,800
103,Anafi AI,3,898
104,Mini 3 Pro,1,249
```

### Тестові набори та утиліти

```lisp
(defun test-database-operations ()
  (let* ((selector-drones (select "drones.csv"))
         (selector-mfg (select "manufacturers.csv"))
         (all-drones (funcall selector-drones))
         (dji-drones (funcall selector-drones :manufacturer_id 1))
         (specific-drone (funcall selector-drones :manufacturer_id 1 :weight 249)))
    
    (pretty-print-hashes all-drones)
    (pretty-print-hashes dji-drones)
    (pretty-print-hashes specific-drone)
    
    (print (convert-records dji-drones))
    (write-hashes-to-csv dji-drones "dji_drones.csv")))
```

### Тестування

```lisp
CL-USER> (test-database-operations)
--- 1 ---
  ID: 101
  MODEL: Mavic 3
  MANUFACTURER_ID: 1
  WEIGHT: 895
--- 2 ---
  ID: 102
  MODEL: Skydio 2+
  MANUFACTURER_ID: 2
  WEIGHT: 800
--- 3 ---
  ID: 103
  MODEL: Anafi AI
  MANUFACTURER_ID: 3
  WEIGHT: 898
--- 4 ---
  ID: 104
  MODEL: Mini 3 Pro
  MANUFACTURER_ID: 1
  WEIGHT: 249
--- 1 ---
  ID: 101
  MODEL: Mavic 3
  MANUFACTURER_ID: 1
  WEIGHT: 895
--- 2 ---
  ID: 104
  MODEL: Mini 3 Pro
  MANUFACTURER_ID: 1
  WEIGHT: 249
--- 1 ---
  ID: 104
  MODEL: Mini 3 Pro
  MANUFACTURER_ID: 1
  WEIGHT: 249

(((ID . 101) (MODEL . "Mavic 3") (MANUFACTURER_ID . 1) (WEIGHT . 895))
 ((ID . 104) (MODEL . "Mini 3 Pro") (MANUFACTURER_ID . 1) (WEIGHT . 249))) 
NIL
```