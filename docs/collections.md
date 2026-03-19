# 📚 Коллекции

## Arrays / Lists

=== "Swift 6"
    ```swift
    // Array (value type с Copy-on-Write)
    var arr: [Int] = [1, 2, 3, 4, 5]
    var empty = [String]()                         // пустой массив
    var filled = [Int](repeating: 0, count: 5)    // [0,0,0,0,0]

    arr.append(6)
    arr.insert(0, at: 0)                           // вставить 0 на позицию 0
    arr.remove(at: 0)                              // удалить по индексу
    arr.removeLast()
    arr.removeFirst()
    arr.removeAll(where: { $0 % 2 == 0 })         // удалить все чётные

    arr.count                                      // количество
    arr.isEmpty                                    // false
    arr.first                                      // Optional(1) — безопасно
    arr.last                                       // Optional
    arr[2]                                         // 3 — краш если out of bounds!
    arr.indices.contains(2) ? arr[2] : 0          // безопасный доступ

    arr.contains(3)                                // true
    arr.contains(where: { $0 > 3 })
    arr.firstIndex(of: 3)                          // Optional(2)

    // Сортировка
    arr.sort()                                     // на месте
    let sorted = arr.sorted()                      // новый массив
    arr.sort { $0 > $1 }                          // убывание
    let sortedBy = arr.sorted(by: { $0 > $1 })

    // Функциональные операции
    arr.filter { $0 > 2 }                         // [3,4,5]
    arr.map { $0 * 2 }                            // [2,4,6,8,10]
    arr.reduce(0, +)                               // 15
    arr.reduce(into: [:]) { $0[$1] = $1 * 2 }    // создать словарь
    arr.compactMap { Int("\($0)") }               // map + убрать nil
    arr.flatMap { [$0, $0 * 10] }                // [1,10,2,20,...]

    // Срезы (ArraySlice — не копирует данные!)
    let slice = arr[1...3]                         // ArraySlice<Int>
    let slice2 = arr.prefix(3)                     // первые 3
    let slice3 = arr.suffix(2)                     // последние 2

    // Zip
    let pairs = zip([1,2,3], ["a","b","c"])        // [(1,"a"),(2,"b"),(3,"c")]
    ```

=== "Kotlin 2.x"
    ```kotlin
    // ВАЖНО различать immutable / mutable!
    val immutable: List<Int> = listOf(1, 2, 3, 4, 5)    // read-only view
    val mutable: MutableList<Int> = mutableListOf(1, 2, 3)

    val list = arrayListOf(1, 2, 3, 4, 5)               // MutableList

    list.add(6)
    list.addAll(listOf(7, 8, 9))
    list.add(0, 0)                                       // вставить 0 на позицию 0
    list.removeAt(0)                                     // удалить по индексу
    list.remove(3)                                       // удалить первый элемент == 3
    list.removeIf { it % 2 == 0 }                       // удалить все чётные

    list.size
    list.isEmpty()
    list.first()                                         // кидает NoSuchElementException!
    list.firstOrNull()                                   // null если пустой — безопасно
    list.last(); list.lastOrNull()
    list.getOrNull(10)                                   // null если out of bounds

    // Сортировка
    list.sort()                                          // на месте
    val sorted = list.sorted()                           // новый список
    list.sortedBy { it }
    list.sortedByDescending { it }
    list.sortedWith(compareBy({ it.length }, { it }))   // многоуровневая

    // Функциональные операции
    list.filter { it > 2 }
    list.map { it * 2 }
    list.reduce { acc, i -> acc + i }
    list.fold(0) { acc, i -> acc + i }                  // как reduce но с начальным значением
    list.sumOf { it }
    list.maxOrNull(); list.minOrNull()
    list.flatMap { listOf(it, it * 10) }
    list.mapNotNull { if (it > 3) it * 2 else null }    // map + filter null

    // Группировка
    list.groupBy { it % 2 == 0 }                        // Map<Boolean, List<Int>>
    list.partition { it > 3 }                           // Pair<List, List>
    list.windowed(3)                                    // скользящее окно
    list.chunked(2)                                     // разбить на части по 2

    // Деструктуризация
    val (first, second, third) = listOf(1, 2, 3)

    // Zip
    val pairs = listOf(1,2,3).zip(listOf("a","b","c"))  // [(1,a),(2,b),(3,c)]
    ```

=== "Python 3.13"
    ```python
    # list (динамический массив, все элементы — ссылки на объекты)
    arr: list[int] = [1, 2, 3, 4, 5]

    arr.append(6)                                  # добавить в конец — O(1)
    arr.insert(0, 0)                               # вставить 0 на позицию 0 — O(n)!
    arr.pop()                                      # удалить последний, вернуть его — O(1)
    arr.pop(0)                                     # удалить по индексу — O(n)!
    arr.remove(3)                                  # удалить первый элемент == 3
    arr.extend([7, 8, 9])                          # добавить несколько

    len(arr)
    arr[0]; arr[-1]                                # первый / последний
    arr[10] if 10 < len(arr) else 0               # безопасный доступ

    3 in arr                                       # True
    arr.index(3)                                   # индекс первого совпадения
    arr.count(3)                                   # сколько раз встречается

    # Сортировка
    arr.sort()                                     # на месте (быстрее)
    sorted_arr = sorted(arr)                       # новый список
    arr.sort(reverse=True)                         # убывание
    arr.sort(key=lambda x: -x)                    # с ключом
    sorted(words, key=str.lower)                  # без учёта регистра

    # Функциональные операции
    filtered = [x for x in arr if x > 2]          # list comprehension (pythonic!)
    mapped   = [x * 2 for x in arr]
    nested   = [x * y for x in range(3) for y in range(3) if x != y]

    # map/filter/reduce (менее pythonic, но работают)
    filtered = list(filter(lambda x: x > 2, arr))
    mapped   = list(map(lambda x: x * 2, arr))
    from functools import reduce
    total    = reduce(lambda acc, x: acc + x, arr, 0)

    # Срезы — мощнейший инструмент
    arr[1:4]                                       # [2,3,4]
    arr[:3]                                        # первые 3
    arr[-2:]                                       # последние 2
    arr[::2]                                       # каждый второй
    arr[::-1]                                      # реверс

    # Группировка
    from itertools import groupby, islice, chain
    grouped = {k: list(v) for k, v in groupby(sorted(arr), key=lambda x: x % 2)}

    # zip — параллельная итерация
    for a, b in zip([1,2,3], ["a","b","c"]):
        print(a, b)

    # enumerate — индекс + значение
    for i, v in enumerate(arr, start=1):
        print(f"{i}: {v}")

    # tuple — неизменяемый "список" (hashable, быстрее list)
    t: tuple[int, str, float] = (1, "hello", 3.14)
    a, b, c = t                                    # распаковка
    first, *rest = t                               # starred unpacking
    ```

---

## Dictionaries / Maps

=== "Swift 6"
    ```swift
    var dict: [String: Int] = ["Alice": 30, "Bob": 25]

    dict["Charlie"] = 35                           // добавить
    dict["Alice"] = 31                             // обновить
    dict.removeValue(forKey: "Bob")
    dict["Unknown"]                                // Optional(nil) — безопасно
    dict["Unknown", default: 0]                    // 0 если нет ключа

    dict.keys; dict.values
    dict.count; dict.isEmpty

    for (key, value) in dict {
        print("\(key): \(value)")
    }
    dict.forEach { print("\($0.key): \($0.value)") }

    // Трансформации
    let doubled  = dict.mapValues { $0 * 2 }
    let filtered = dict.filter { $0.value > 30 }
    let keys     = dict.keys.sorted()

    // Merge (Swift 5.4+)
    dict.merge(["Dave": 22]) { current, _ in current }
    ```

=== "Kotlin 2.x"
    ```kotlin
    val immutableMap = mapOf("Alice" to 30, "Bob" to 25)     // read-only
    val map = mutableMapOf("Alice" to 30, "Bob" to 25)

    map["Charlie"] = 35
    map["Alice"] = 31
    map.remove("Bob")
    map["Unknown"]                                 // null — безопасно
    map.getOrDefault("Unknown", 0)
    map.getOrPut("Dave") { 20 }                   // добавить если нет, вернуть значение

    for ((key, value) in map) { println("$key: $value") }
    map.forEach { (k, v) -> println("$k: $v") }

    map.mapValues { it.value * 2 }
    map.filter { it.value > 30 }
    map.keys.sorted()
    map.toSortedMap()
    map.entries.sortedBy { it.value }

    // Merge
    map.putAll(mapOf("Eve" to 28))
    ```

=== "Python 3.13"
    ```python
    d: dict[str, int] = {"Alice": 30, "Bob": 25}

    d["Charlie"] = 35
    d["Alice"] = 31
    del d["Bob"]
    d.get("Unknown")                               # None — безопасно
    d.get("Unknown", 0)                            # 0 если нет ключа
    d.setdefault("Dave", 20)                       # добавить если нет
    d.pop("Alice", None)                           # удалить безопасно

    d.keys(); d.values(); d.items()

    for key, value in d.items():
        print(f"{key}: {value}")

    # Dict comprehension
    doubled  = {k: v * 2 for k, v in d.items()}
    filtered = {k: v for k, v in d.items() if v > 25}

    # Merge (Python 3.9+)
    merged = d | {"Eve": 28}                       # новый dict
    d |= {"Eve": 28}                               # update на месте

    # Сортировка по значению
    sorted_by_val = dict(sorted(d.items(), key=lambda x: x[1]))

    # Counter — специализированный dict для подсчёта
    from collections import Counter
    c = Counter(["a","b","a","c","a","b"])
    c.most_common(2)                               # [('a', 3), ('b', 2)]

    # defaultdict — автосоздание значения при отсутствии ключа
    from collections import defaultdict
    graph: defaultdict[str, list[str]] = defaultdict(list)
    graph["A"].append("B")
    ```

---

## Sets

=== "Swift 6"
    ```swift
    var s: Set<Int> = [1, 2, 3, 4]

    s.insert(5)
    s.remove(1)
    s.contains(3)                                  // true

    let a: Set = [1, 2, 3]
    let b: Set = [2, 3, 4]
    a.union(b)                                     // {1,2,3,4}
    a.intersection(b)                              // {2,3}
    a.subtracting(b)                               // {1}
    a.symmetricDifference(b)                       // {1,4}
    a.isSubset(of: b)                              // false
    a.isDisjoint(with: b)                          // false
    ```

=== "Kotlin 2.x"
    ```kotlin
    val immutableSet = setOf(1, 2, 3, 4)
    val set = mutableSetOf(1, 2, 3, 4)

    set.add(5); set.remove(1); 3 in set

    val a = setOf(1, 2, 3)
    val b = setOf(2, 3, 4)
    a union b                                      // {1,2,3,4}
    a intersect b                                  // {2,3}
    a subtract b                                   // {1}
    ```

=== "Python 3.13"
    ```python
    s: set[int] = {1, 2, 3, 4}

    s.add(5)
    s.remove(1)                                    # KeyError если нет
    s.discard(100)                                 # безопасно
    3 in s

    a, b = {1, 2, 3}, {2, 3, 4}
    a | b                                          # {1,2,3,4}  union
    a & b                                          # {2,3}      intersection
    a - b                                          # {1}        difference
    a ^ b                                          # {1,4}      symmetric difference
    a <= b                                         # issubset
    a.isdisjoint(b)                               # False

    # frozenset — неизменяемое и хэшируемое (можно использовать как ключ dict)
    fs = frozenset([1, 2, 3])
    d = {fs: "value"}                              # OK!
    ```
