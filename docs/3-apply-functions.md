# Функции семейства apply

## Описание
На предыдущем уроке мы изучили всевозможные варианты циклов в R, но вам всё время говорят о том, что не надо использовать циклы в R. Возникает вопрос, так что же использовать вместо циклов? На самом деле есть альтернатива в виде функционалов. В этом уроке мы разберёмся с функционалами в базовом синтаксисе R, которые реализованы в семействе функций `apply()`.

Функционал - это функция, которая перебирает элементы объекта применяя последовательно к каждому элементу заданную функцию. 

## Видео
<iframe width="560" height="315" src="https://www.youtube.com/embed/9uitTb_RWV0?enablejsapi=1" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Тайм коды
00:00 Вступление.<Br>
00:48 Какие функции входят в семейство `apply`.<Br>
02:22 Функция `apply()`.<Br>
07:57 Передача дополнительных аргументов в применяемую внутри `apply()` функцию.<Br>
09:05 Функции `lapply()`, `sapply()` и `vapply()`.<Br>
12:09 Как использовать  самописную функцию внутри функций семейства `apply`.<Br>
13:23 Пример чтения данных из множества csv файла функцией `lapply()`.<Br>
15:40 Функция `mapply()`.<Br>
18:00 Заключение.<Br>

## Код

```r
# apply family

# пример с циклом ---------------------------------------------------------
# строки
for ( x in seq_along(1:nrow(mtcars)) ) {
  cat(rownames(mtcars[x,]), ":", sum(mtcars[x,]), "\n")
}

# столбцы
col_num <- 1

for ( x in mtcars ) {
  cat(names(mtcars)[col_num], ":", sum(x), "\n")
  col_num <- col_num + 1
}

# apply -------------------------------------------------------------------
# 1 - строки
# 2 - столюцы
apply(mtcars, 1, sum)
apply(mtcars, 2, sum)

sum(mtcars[3, ])
sum(mtcars[ ,3])
# row operation -----------------------------------------------------------
rowSums(mtcars)
rowMeans(mtcars)
# передача дополнительных аргументов --------------------------------------
apply(mtcars, 2, quantile, probs = 0.25)
quantile(mtcars[, 3], probs = 0.25)

# lapply ------------------------------------------------------------------
values <- list(
  x = c(4, 6, 1),
  y = c(5, 10, 1, 23, 4),
  z = c(2, 5, 6, 7)
)

lapply(values, sum)
sapply(values, sum)
vapply(values, sum, FUN.VALUE = 7)

# lapply с самописной функцией --------------------------------------------
fl <- function(x) {
  num_elements <- length(x)
  return(x[1] + x[num_elements])
}

lapply(values, fl)


# пример чтения файлов ----------------------------------------------------
directory <- 'C:/Users/Alsey/Documents/docs/'
files <- dir(path = directory, pattern = '\\.csv$')
all_data <- list()

# цикл 
for ( file in files ) {
  data <- read.csv(paste0(directory, file))
  all_data <- append(all_data, list(data))
}

dplyr::bind_rows(all_data)

# lapply
file_paths <- paste0(directory, files)
all_data <- lapply(file_paths, read.csv)
dplyr::bind_rows(all_data)


# mapply ------------------------------------------------------------------
mapply(rep, 1:4, times=4:1)
mapply(rep, times = 1:4, x = 4:1)
```

## Презентация
<iframe src="https://www.slideshare.net/slideshow/embed_code/key/EgI6pMw0iQfhes" width="595" height="485" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="https://www.slideshare.net/ssuser459d25/apply-250849873" title="Функции семейства apply" target="_blank">Функции семейства apply</a> </strong> from <strong><a href="https://www.slideshare.net/ssuser459d25" target="_blank">Алексей Селезнёв</a></strong> </div>

## Тест
<iframe id="otp_wgt_tpvame3oxrnw2" src="https://onlinetestpad.com/tpvame3oxrnw2" frameborder="0" style="width:100%;" onload="var f = document.getElementById('otp_wgt_tpvame3oxrnw2'); var h = 0; var listener = function (event) { if (event.origin.indexOf('onlinetestpad') == -1) { return; }; h = parseInt(event.data); if (!isNaN(h)) f.style.height = h + 'px'; }; function addEvent(elem, evnt, func) { if (elem.addEventListener) { elem.addEventListener(evnt, func, false); } else if (elem.attachEvent) { elem.attachEvent('on' + evnt, func); } else { elem['on' + evnt] = func; } }; addEvent(window, 'message', listener);" scrolling="no">
</iframe>

## Дополнительные материалы
* [Статья "Векторизованные вычисления в R с использованием apply-функций"](https://r-analytics.blogspot.com/2012/11/r-apply.html) (Сергей Мастицкий).
