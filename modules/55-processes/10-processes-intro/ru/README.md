
Весь Elixir код запускается в рамках *процесса*. Важно понимать, что процесс в Elixir не является аналогией процесса операционной системы.

Процессы в Elixir являются зелеными потоками (green threads) - это более легковесные процессы, чем в операционной системе, которые выполняются в рамках виртуальной машины языка программирования. В случае Elixir это BEAM. Внутри виртуальной машины выделяются отдельные планировщики, как и в операционной системе, которые следят за тем, чтобы каждый процесс работал определенный квант времени, например 4 миллисекунды. Благодаря такой отлаженной работы планировщиков нагрузка на процессор распределяется более плавно относительно каждого из ядер, то есть ситуации, когда одно ядро нагружено сильнее другого возникают намного реже. По сути, BEAM реализует на практике вытесняющую многозадачность.

Процессы изолированы и выполняются *конкурентно* относительно друг друга. Общаются процессы между собой через *сообщения*, забавный факт, что первая предложенная Аланом Кеем модель ООП как раз описывалась как множество объектов с внутренним состоянием, которые коммуницируют между собой через сообщения.

Из-за легковесности процессов, зачастую в Elixir приложениях запущенны сотни, а то и тысячи процессов. Для создания процесса используем функцию `spawn`:

```elixir
spawn(fn -> 2 * 2 end)
# => #PID<0.43.0>

# создадим 100 тысяч процессов
processes = 0..100_000 |> Enum.map(fn _ -> spawn(fn -> 2 + 2 end) end)
# => много иднтификаторов процессов

hd(processes)
# => #PID<0.5249.9>
List.last(processes)
# => #PID<0.6945.12>
```

Функция `spawn` принимает аргументом функцию, которую запускает в отдельном процессе. Заметьте, что вызов `spawn` возвращает не результат выполнения переданной функции, а идентификатор процесса (process identifier) или ПИД (PID). Процесс, который был создан в предыдущем примере, после выполнения функции завершился.

```elixir
pid = spawn(fn -> 2 * 2 end)
# => #PID<0.44.0>

Process.alive?(pid)
# => false
```

С помощью функции `self` мы можем узнать идентификатор нынешнего процесса, в котором была вызвана функция:

```elixir
self()
# => #PID<0.41.0>

Process.alive?(self())
# => true
```