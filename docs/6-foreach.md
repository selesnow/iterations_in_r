# Многопоточность в R

## Описание
Давайте представим ситуацию, что вам необходимо доствить 8 адресатам посылки. Если вы будете доставлять их одним курьером, то ему придётся по очереди посетить все 8 адресов, собрать подписи в качестве подтверждения о получении посылки, и принести вам подписанные документы. но если у вас в распоряжении будет 4 курьера, то вы сможете распределить каждому курьеру всего по 2 адреса, и процесс доставки займёт в 4 раза меньше времени. 

Ок, а при чём тут вообще курьеры спросите вы. Во всех предыдущих уроках мы выполняли итерирование по элементов объектов в последовательном режиме, т.е. использовали одного курьера. Это преемлемый способ итерирования, но не самый эффективный. В этом уроке мы с вами разберёмся с тем, как задействовать сразу 4ёх курьеров, т.е. выполнять итерации в параллеьном, многопоточном режиме.

Так же мы можем сделать этот процесс ещё более эффективным, если будем не рандомно раздавать курьерам адресатов, а например распредим каждому курьеру по одному району, это балансировка нагрузки, её мы тоже затронем в этом уроке.

## Видео
<iframe width="560" height="315" src="https://www.youtube.com/embed/im7tKu9XgB0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Тайм коды
00:00 Вступление.<Br>
00:51 Что такое многопоточность.<Br>
02:20 Какие пакеты мы будем использовать в ходе урока.<Br>
03:25 Используем `foreach` в последовательном режиме.<Br>
07:42 Аргументы конструкции `foreach.`<Br>
10:05 Управление объединением результатов итераций цикла `foreach.`<Br>
11:05 Выполнение `foreach` в многопоточном режиме.<Br>
12:41 Схема реализации многопоточности.<Br>
13:52 Возвращение к последовательному выполнению и ID процесса.<Br>
14:56 Бекенды к `foreach.`<Br>
15:38 Оператор `%dorng%`.<Br>
18:10 Параллельная реализация функций семейства `apply.`<Br>
20:52 Список функций пакетов `parallel` и `pbapply.`<Br>
21:54 Пакет `furrr`.<Br>
23:10 Соответствие функций пакета `purrr` и `furrr`.<Br>
23:50 Заключение.<Br>

## Код

```r
# многопоточные циклы -----------------------------------------------------
# install.packages("doSNOW")
# library(doSNOW)
# library(doParallel)
library(doFuture)

# функция длительного выполнения
pause <- function(min = 1, max = 3) {
  ptime <- runif(1, min, max)

  Sys.sleep(ptime)

  out <- list(
    pid = Sys.getpid(),
    pause_sec = ptime
  )
}

test <- pause()

# используем foreach 
# итерируемся сразу по двум объектам
system.time (
  {test2 <- foreach(min = 1:3, max = 2:4) %do% pause(min, max)}
)

# сумма длительностей пауз
sum(sapply(test2, '[[', i = 'pause_sec'))

# меняем функцию собирающую результаты каждой итерации
test3 <- foreach(min = 1:3, max = 2:4, .combine = dplyr::bind_rows) %do% pause(min, max)

# параллельный режим выполнения
# создаём кластер из четырёх ядер
#cl <- makeCluster(4)
#registerDoSNOW(cl)

options(future.rng.onMisuse = "ignore")
registerDoFuture()
plan('multisession', workers = 3)

# выполняем тот же код но в параллельном режиме
system.time (
  {
    par_test1 <- 
      foreach(min = 1:3, max = 2:4, .combine = dplyr::bind_rows) %dopar% {
      pause(min, max)
    }
  }
)

# останавливаем кластер
plan('sequential')

par_test1


# многопоточный вариант функций apply -------------------------------------

library(pbapply)
library(parallel)

# создаём кластер из четырёх ядер
cl <- makeCluster(3)

# пример с pbapply
par_test2 <- pblapply(rep(1, 3), FUN = pause, max = 3, cl = cl)
# пример с parallel
par_test3 <- parLapply(rep(1, 3), fun = pause, max = 3, cl = cl)

# останавливаем кластер
stopCluster(cl)

# многопоточный purrr -----------------------------------------------------
library(furrr)

plan('multisession', workers = 3)

par_test4 <- future_map2(1:3, 2:4, pause)

# останавливаем кластер
plan('sequential')
```

## Презентация
<iframe src="https://www.slideshare.net/slideshow/embed_code/key/LqdDlr12lIhmuG" width="595" height="485" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="https://www.slideshare.net/ssuser459d25/r-250850079" title="Многопоточность в R" target="_blank">Многопоточность в R</a> </strong> from <strong><a href="https://www.slideshare.net/ssuser459d25" target="_blank">Алексей Селезнёв</a></strong> </div>

## Тест
<iframe id="otp_wgt_b46rrysox22bc" src="https://onlinetestpad.com/b46rrysox22bc" frameborder="0" style="width:100%;" onload="var f = document.getElementById('otp_wgt_b46rrysox22bc'); var h = 0; var listener = function (event) { if (event.origin.indexOf('onlinetestpad') == -1) { return; }; h = parseInt(event.data); if (!isNaN(h)) f.style.height = h + 'px'; }; function addEvent(elem, evnt, func) { if (elem.addEventListener) { elem.addEventListener(evnt, func, false); } else if (elem.attachEvent) { elem.attachEvent('on' + evnt, func); } else { elem['on' + evnt] = func; } }; addEvent(window, 'message', listener);" scrolling="no">
</iframe>

## Дополнительные материалы
* Статья ["Как ускорить работу с API на языке R с помощью параллельных вычислений, на примере API Яндекс.Директ (Часть 1)"](https://habr.com/ru/post/437078/)
