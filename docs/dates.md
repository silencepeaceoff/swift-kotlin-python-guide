# 📅 Даты и время

=== "Swift 6"
    ```swift
    import Foundation

    // Текущая дата
    let now = Date()
    let nowInstant = Date.now                      // Swift 5.7+

    // Компоненты даты
    let calendar = Calendar.current
    let components = calendar.dateComponents([.year, .month, .day, .hour], from: now)
    let year  = components.year!
    let month = components.month!

    // Создание даты из компонентов
    var dc = DateComponents()
    dc.year = 2026; dc.month = 3; dc.day = 15; dc.hour = 10
    dc.timeZone = TimeZone(identifier: "Europe/Berlin")
    let specificDate = calendar.date(from: dc)!

    // Форматирование (FormatStyle Swift 5.5+)
    let formatted = now.formatted(
        .dateTime.day().month(.wide).year()
    )
    // "March 19, 2026"

    let custom = now.formatted(
        Date.FormatStyle()
            .year(.defaultDigits)
            .month(.twoDigits)
            .day(.twoDigits)
            .hour(.twoDigits(amPM: .omitted))
            .minute(.twoDigits)
    )

    // ISO 8601
    let iso = now.ISO8601Format()

    // Парсинг
    let parsed = try Date("2026-03-15T10:30:00Z", strategy: .iso8601)

    // Duration / Arithmetic
    let tomorrow = calendar.date(byAdding: .day, value: 1, to: now)!
    let nextWeek = calendar.date(byAdding: .weekOfYear, value: 1, to: now)!
    let diff = calendar.dateComponents([.day, .hour], from: now, to: tomorrow)

    // Clock API (Swift 5.7+)
    let clock = ContinuousClock()
    let elapsed = try await clock.measure {
        try await Task.sleep(for: .seconds(1))
    }
    print("Elapsed: \(elapsed)")                   // "Elapsed: 1.0 seconds"
    ```

=== "Kotlin 2.x"
    ```kotlin
    import java.time.*
    import java.time.format.DateTimeFormatter
    import java.time.temporal.ChronoUnit

    // Текущая дата/время
    val now      = LocalDateTime.now()
    val today    = LocalDate.now()
    val time     = LocalTime.now()
    val instant  = Instant.now()                   // UTC timestamp

    // С timezone
    val berlinNow = ZonedDateTime.now(ZoneId.of("Europe/Berlin"))

    // Создание
    val specific = LocalDateTime.of(2026, Month.MARCH, 15, 10, 30)
    val date     = LocalDate.of(2026, 3, 15)

    // Компоненты
    val year  = now.year
    val month = now.month                          // Month enum
    val day   = now.dayOfMonth
    val dow   = now.dayOfWeek                      // DayOfWeek enum

    // Форматирование
    val formatted = now.format(DateTimeFormatter.ofPattern("dd.MM.yyyy HH:mm"))
    val iso       = now.format(DateTimeFormatter.ISO_LOCAL_DATE_TIME)

    // Парсинг
    val parsed = LocalDate.parse("2026-03-15")
    val parsed2 = LocalDateTime.parse("15.03.2026 10:30",
                    DateTimeFormatter.ofPattern("dd.MM.yyyy HH:mm"))

    // Арифметика
    val tomorrow = today.plusDays(1)
    val nextWeek = today.plusWeeks(1)
    val ago      = today.minusMonths(3)

    // Duration (для время) и Period (для даты)
    val duration = Duration.between(now, now.plusHours(5))
    val period   = Period.between(date, today)

    val daysBetween = ChronoUnit.DAYS.between(date, today)

    // kotlinx-datetime (multiplatform)
    import kotlinx.datetime.*

    val nowKx = Clock.System.now()                 // Instant
    val tz    = TimeZone.of("Europe/Berlin")
    val local = nowKx.toLocalDateTime(tz)
    ```

=== "Python 3.13"
    ```python
    from datetime import datetime, date, time, timedelta, timezone
    from zoneinfo import ZoneInfo                  # Python 3.9+ (встроен)
    from dateutil.relativedelta import relativedelta  # pip install python-dateutil

    # Текущая дата/время
    now   = datetime.now()
    today = date.today()
    utc   = datetime.now(timezone.utc)

    # С timezone
    berlin = datetime.now(ZoneInfo("Europe/Berlin"))

    # Создание
    specific = datetime(2026, 3, 15, 10, 30, 0, tzinfo=ZoneInfo("Europe/Berlin"))

    # Компоненты
    year  = now.year
    month = now.month
    day   = now.day
    dow   = now.weekday()                          # 0=Пн, 6=Вс
    dow2  = now.isoweekday()                       # 1=Пн, 7=Вс

    # Форматирование
    formatted = now.strftime("%d.%m.%Y %H:%M")    # "19.03.2026 21:34"
    iso       = now.isoformat()                    # "2026-03-19T21:34:59"

    # Парсинг
    parsed = datetime.strptime("15.03.2026 10:30", "%d.%m.%Y %H:%M")
    parsed2 = date.fromisoformat("2026-03-15")
    parsed3 = datetime.fromisoformat("2026-03-15T10:30:00+01:00")  # 3.11+

    # Арифметика
    tomorrow  = today + timedelta(days=1)
    next_week = today + timedelta(weeks=1)
    diff      = datetime(2026, 12, 31) - now       # timedelta

    # relativedelta — месяцы и годы (timedelta не умеет)
    next_month = today + relativedelta(months=1)
    next_year  = today + relativedelta(years=1)
    age = relativedelta(today, date(1990, 1, 1))
    print(f"{age.years} лет, {age.months} месяцев")

    # Timestamp (Unix epoch)
    ts = now.timestamp()                           # float
    from_ts = datetime.fromtimestamp(ts, tz=timezone.utc)

    # Performance timing
    import time
    start = time.perf_counter()
    heavy_work()
    elapsed = time.perf_counter() - start
    print(f"{elapsed:.4f}s")

    # monotonic (не зависит от системных часов)
    mono_start = time.monotonic()
    ```
