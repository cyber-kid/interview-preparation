# Dates
* [Creating Dates and Times](#creating-dates-and-times)
* [Formatting dates](#formatting-dates)
* [Manipulating Dates and Times](#manipulating-dates-and-times)
* [Working with Periods](#working-with-periods)
* [Working with Duration](#working-with-duration)
* [Working with Instant](#working-with-instant)
#### Creating Dates and Times
The date and time classes are in the **java.time** package. The classes defined in this package represent the principle date-time concepts, including instants, durations, dates, times, time-zones and periods. All the classes of this package are immutable and thread-safe.

When working with dates and times, the first thing to do is to decide how much information you need.
* **LocalDate** Contains just a date—no time and no time zone. A good example of LocalDate is your birthday this year. It is your birthday for a full day, regardless of what time it is.
* **LocalTime** Contains just a time—no date and no time zone. A good example of LocalTime is midnight. It is midnight at the same time every day.
* **LocalDateTime** Contains both a date and time but no time zone. A good example of LocalDateTime is “the stroke of midnight on New Year’s Eve.” Midnight on January 2 isn’t nearly as special, making the date relatively unimportant, and clearly an hour after midnight isn’t as special either.
* **ZonedDateTime** Contains a date, time, and time zone. A good example of ZonedDateTime is “a conference call at 9:00 a.m. EST.” If you live in California, you’ll have to get up really early since the call is at 6:00 a.m. local time!

Each of the four classes has a static method called **now()**, which gives the current date and time.

To create a date use:
* public static LocalDate.of(int year, int month, int dayOfMonth)
* public static LocalDate.of(int year, Month month, int dayOfMonth)

To create a time use:
* public static LocalTime.of(int hour, int minute)
* public static LocalTime.of(int hour, int minute, int second)
* public static LocalTime.of(int hour, int minute, int second, int nanos)

You can combine dates and times into one object:
* public static LocalDateTime.of(int year, int month, int dayOfMonth, int hour, int minute)
* public static LocalDateTime.of(int year, int month, int dayOfMonth, int hour, int minute, int second)
* public static LocalDateTime.of(int year, int month, int dayOfMonth, int hour, int minute, int second, int nanos)
* public static LocalDateTime.of(int year, Month month, int dayOfMonth, int hour, int minute)
* public static LocalDateTime.of(int year, Month month, int dayOfMonth, int hour, int minute, int second)
* public static LocalDateTime.of(int year, Month month, int dayOfMonth, int hour, int minute, int second, int nanos)
* public static LocalDateTime.of(LocalDate date, LocalTime time)

To create a time with time zone use:
* public static ZonedDateTime.of(int year, int month, int dayOfMonth, int hour, int minute, int second, int nanos, ZoneId zone)
* public static ZonedDateTime.of(LocalDate date, LocalTime time, ZoneId zone)
* public static ZonedDateTime.of(LocalDateTime dateTime, ZoneId zone)

To get ZoneId use:
* ZoneId zone = ZoneId.of("US/Eastern");

To get default ZoneId use:
* public static ZoneId systemDefault();

To get all available ZoneId use:
* public static Set<String> getAvailableZoneIds()
#### Formatting dates
You can use **DateTimeFormatter** to format date and time. **DateTimeFormatter** is in the package **java.time.format**.
```java
LocalDate date = LocalDate.of(2020, Month.JANUARY, 20);
LocalTime time = LocalTime.of(11, 12, 34);
LocalDateTime dateTime = LocalDateTime.of(date, time);
System.out.println(date.format(DateTimeFormatter.ISO_LOCAL_DATE));
System.out.println(time.format(DateTimeFormatter.ISO_LOCAL_TIME));
System.out.println(dateTime.format(DateTimeFormatter.ISO_LOCAL_DATE_TIME));
```
There are some predefined formats for default locale:
```java
DateTimeFormatter shortDateTime =
DateTimeFormatter.ofLocalizedDate(FormatStyle.SHORT);
System.out.println(shortDateTime.format(dateTime)); // 1/20/20
System.out.println(shortDateTime.format(date)); // 1/20/20
System.out.println(shortDateTime.format(time)); // UnsupportedTemporalTypeException
```
The **format()** method is declared on both the formatter objects and the date/time objects, allowing you to reference the objects in either order.

Table shows the legal and illegal localized formatting methods.

DateTimeFormatter f = DateTimeFormatter.____(FormatStyle.SHORT); | Calling f.format(localDate); | Calling f.format(localDateTime) or (zonedDateTime) | Calling f.format(localTime);
---------------------------------------------------------------- | ---------------------------- | -------------------------------------------------- | ----------------------------
ofLocalizedDate | Legal—shows whole object | Legal—shows just date part | Throws runtime exception 
ofLocalizedDateTime | Throws runtime exception | Legal—shows whole object | Throws runtime exception 
ofLocalizedTime | Throws runtime exception | Legal—shows just time part | Legal—shows whole object

#### Manipulating Dates and Times
The date and time classes are immutable.
Examples:
(add)
```java
LocalDate date = LocalDate.of(2014, Month.JANUARY, 20);
date = date.plusDays(2);
date = date.plusWeeks(1);
date = date.plusMonths(1);
date = date.plusYears(5);
```
(minus)
```java
LocalDate date = LocalDate.of(2020, Month.JANUARY, 20);
LocalTime time = LocalTime.of(5, 15);
LocalDateTime dateTime = LocalDateTime.of(date, time);
dateTime = dateTime.minusDays(1);
dateTime = dateTime.minusHours(10);
dateTime = dateTime.plusHours(10);
dateTime = dateTime.minusSeconds(30);
dateTime = dateTime.plusSeconds(30);
```
It is common for date and time methods to be chained:
```java
LocalDate date = LocalDate.of(2020, Month.JANUARY, 20);
LocalTime time = LocalTime.of(5, 15);
LocalDateTime dateTime = LocalDateTime.of(date, time) .minusDays(1).minusHours(10).minusSeconds(30);
```
#### Working with Periods
A Period is a day or more of time. There are five ways to create a Period class:
* Period annually = Period.ofYears(1);
* Period quarterly = Period.ofMonths(3);
* Period everyThreeWeeks = Period.ofWeeks(3);
* Period everyOtherDay = Period.ofDays(2);
* Period everyYearAndAWeek = Period.of(1, 0, 7);

There’s one catch. You cannot chain methods when creating a Period.

If you attempt to add a month to an object that has only a time. This won’t work. Java throws an exception and complains that we attempted to use an Unsupported unit: Months.
#### Working with Duration
For Duration, you can specify the number of days, hours, minutes, seconds, or nanoseconds.We can create a Duration using a number of different granularities:
* Duration daily = Duration.ofDays(1);
* Duration hourly = Duration.ofHours(1);
* Duration everyMinute = Duration.ofMinutes(1);
* Duration everyTenSeconds = Duration.ofSeconds(10);
* Duration everyMilli = Duration.ofMillis(1);
* Duration everyNano = Duration.ofNanos(1);

Duration doesn’t have a constructor that takes multiple units like Period does. If you want something to happen every hour and a half, you would specify 90 minutes.
Duration includes another more generic factory method. It takes a number and a TemporalUnit. The idea is, say, something like “5 seconds.” However, TemporalUnit is an interface. At the moment, there is only one implementation named ChronoUnit.
* Duration daily = Duration.of(1, ChronoUnit.DAYS);
* Duration hourly = Duration.of(1, ChronoUnit.HOURS);
* Duration everyMinute = Duration.of(1, ChronoUnit.MINUTES);
* Duration everyTenSeconds = Duration.of(10, ChronoUnit.SECONDS);
* Duration everyMilli = Duration.of(1, ChronoUnit.MILLIS);
* Duration everyNano = Duration.of(1, ChronoUnit.NANOS);

Since we are working with a LocalDate, we are required to use Period. Duration has time units in it, even if we don’t see them and they are meant only for objects with time.
#### Working with Instant
The Instant class represents a specific moment in time in the GMT time zone. Suppose that you want to run a timer:
```java
Instant now = Instant.now();
// do something time consuming
Instant later = Instant.now();
Duration duration = Duration.between(now, later);
System.out.println(duration.toMillis());
```
Using that Instant, you can do math. Instant allows you to add any unit day or smaller, for example:
* Instant nextDay = instant.plus(1, ChronoUnit.DAYS);
* Instant nextHour = instant.plus(1, ChronoUnit.HOURS);
* Instant nextWeek = instant.plus(1, ChronoUnit.WEEKS); // exception

It’s weird that an Instant displays a year and month while preventing you from doing math with those fields. Unfortunately, you need to memorize this fact.

Remember
1. Instant  is a point on Java time line. This timeline start from  from the first second of January 1, 1970 (1970-01-01T00:00:00Z) also called the EPOCH. Note that there is no time zone here. You may think of it as "Java Time Zone" and it matches with GMT or UTC time zone. That means, 1PM in Java time zone will be same as 1 PM in GMT or UTC time zone.
2. Once created, an Instant cannot be modified. Its methods such as plus and minus return a new Instant object.
3. Instant works with time (instead of dates), so you can use Duration instance to create new Instants.
4. LocalDateTime is a time in a given time zone. (But remember that an instance of LocalDateTime itself does not store the zone information!). You can, therefore, use an Instant and a time zone to create a LocalDateTime object. 
5. Whenever you convert an Instant to a LocalDateTime using a time zone, just add or substract the GMT offset of the time zone i.e. if the time zone is GMT+2, add 2 hours and if the time zone is GMT-2, substract two hours. For example, you can yourself this question - if it is 1PM (for example) here in London, which is in GMT, then what would be the time in New York (for example), which is in GMT-4 (or 5, depending on whether the date lies when Day Light Savings time is on or not). The answer would be 1PM - 4 i.e. 9AM. 