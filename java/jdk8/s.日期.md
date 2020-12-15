```java
public static void main(String[] args) {

    //当前日期
    LocalDate localDate = LocalDate.now();
    System.out.println(localDate);
    System.out.println(localDate.getYear() + "," + localDate.getMonthValue() + "," + localDate.getDayOfMonth());
    System.out.println("---------------------");
    //年月日构造日期
    LocalDate localDate1 = LocalDate.of(2014, 2,4 );
    System.out.println(localDate1);
    //判断是否在之前、之后、相等
    System.out.println(localDate.isAfter(localDate1));
    System.out.println(localDate.isBefore(localDate1));
    System.out.println(localDate.equals(localDate1));
    System.out.println("---------------------");
    //两周之后日期
    LocalDate localDate2  = localDate.plus(2, ChronoUnit.WEEKS);
    System.out.println(localDate2);
    System.out.println("---------------------");
    //减去两个月
    LocalDate localDate3  = localDate.minus(2, ChronoUnit.MONTHS);
    System.out.println(localDate3);
    System.out.println("---------------------");

    //月-日
    MonthDay monthDay = MonthDay.from(localDate1);
    MonthDay monthDay1 = MonthDay.of(2, 4);
    System.out.println(monthDay.equals(monthDay1));
    System.out.println("---------------------");

    //年-月
    YearMonth yearMonth = YearMonth.now();
    YearMonth yearMonth1 = YearMonth.of(2019,2 );
    System.out.println(yearMonth);
    System.out.println(yearMonth.lengthOfMonth());
    System.out.println(yearMonth.lengthOfYear());
    System.out.println(yearMonth.isLeapYear());
    System.out.println("---------------------");

    //time时间
    LocalTime time = LocalTime.now();
    System.out.println(time);
    LocalTime time1 = time.plusHours(2).plusMinutes(23);
    System.out.println(time1);
    System.out.println("---------------------");

    //当前时刻
    Clock clock = Clock.systemDefaultZone();
    System.out.println(clock.millis());
    System.out.println("---------------------");

    //时区
    Set<String> availableZoneIds = ZoneId.getAvailableZoneIds();
    System.out.println(availableZoneIds);
    ZoneId zoneId = ZoneId.of("Asia/Shanghai");
    //时间、带时区时间
    LocalDateTime localDateTime = LocalDateTime.now();
    System.out.println(localDateTime);
    ZonedDateTime zonedDateTime = ZonedDateTime.of(localDateTime,zoneId );
    System.out.println(zonedDateTime);
    System.out.println("---------------------");

    //时间转化
    // 2017-01-05  // 2017-01-05  // 今天是：2017年 一月 05日 星期四
    String strDate2 = localDateTime.format(DateTimeFormatter.ISO_LOCAL_DATE);
    String strDate4 = localDateTime.format(DateTimeFormatter.ofPattern("yyyy-MM-dd"));
    String strDate5 = localDateTime.format(DateTimeFormatter.ofPattern("今天是：yyyy年 MM dd日", Locale.CHINESE));

    System.out.println(LocalDateTime.parse("2020-02-18 02:12:21", DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")));
}
```